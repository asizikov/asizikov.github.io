---
layout: post
title: Docker image tests
image: /images/2020-09-container-tests/dive.png
---



Today we're going to set up a CI/CD GitHub Action with a Container Structure Test step which will help us to enforce the certain quality policies for the images which we build and ship.

It's a good idea to review your docker images. First of all, it can save time, disk space, and money. 

When our images are lean the build time is reduced as well as the pull and startup time. Local Docker cache takes less space. And as a cherry on the cake, we pay less for our cloud storage and egress/cross-region/cross-az traffic. 

A win-win-win situation.

We also should take care of the content of our images because we don't need to provide a broader attack surface. Fewer tools installed means fewer opportunities for those zero-days exploits.

And last, but not least everyone who can pull our can inspect them (tools like [dive](https://github.com/wagoodman/dive) can make this process easy as pie).

![Dive PNG screenshot](/images/2020-09-container-tests/dive.png)

So it makes sense to reject images with sensitive data/information baked in the layer.

## Container Structure Test

There are several tools that are available to help us enforce policies on the images that we build and deploy. One of them is a framework called [Container Structure Tests](https://github.com/GoogleContainerTools/container-structure-test)

The Container Structure Tests provide a framework to validate the structure of a container image. These tests can be used to check the output of commands in an image, as well as verify metadata and contents of the filesystem.

Tests can be run either through a standalone binary or through a Docker image.

```bash
container-structure-test test --image gcr.io/registry/image:latest \
--config config.yaml
```

Let's check a few examples. 

## Bad Docker image

I'm going to use [this repository](https://github.com/asizikov/asp-net-container) as an example. Feel free to fork it and play with it yourself.

Let's review the docker file first: 

{% gist 05bc249e0c6ecbd66a85b9da89427c20 %}

Everything is wrong about it: 

* All the files are transferred to the docker daemon
* .NET Core SDK is shipped to production
* Unit tests are shipped as well
* Many useless layers 
* Large image size

```bash
$ docker images
REPOSITORY           TAG       SIZE
asp-net-container    latest    850MB
```

850 MB for a super basic echo application. Let's get those things fixed.

## Local setup

At first, I'm going to install container-structure-tests locally.

### Linux

```bash
curl -LO \
https://storage.googleapis.com/container-structure-test/latest/container-structure-test-linux-amd64 \
&& chmod +x container-structure-test-linux-amd64 && \ 
sudo mv container-structure-test-linux-amd64 /usr/local/bin/container-structure-test
```

### MacOS

```bash
curl -LO \
https://storage.googleapis.com/container-structure-test/latest/container-structure-test-darwin-amd64 && \ 
chmod +x container-structure-test-darwin-amd64 && \ 
sudo mv container-structure-test-darwin-amd64 /usr/local/bin/container-structure-test
```

And let's create `container-structure-tests.yaml` file with the following content:

{% gist 5b35287e09341987f8abd9d4ebceec14 %}

This test verifies if we have `.git` directory copied over to the `/app` working directory.

And now we can run this (I assume that we already have an image built from our Dockerfile)

```bash
container-structure-test test \
        --image asp-net-container:latest \
        --config container-structure-tests.yaml 
```

![The first test failed](/images/2020-09-container-tests/git-test.png)

This is something that can be fixed with a good `.dockerignore` file:

{%gist 146eaf6fc5a7eb20443abca93a6e3c8a %}

When I measure the difference on the CI (GitHub Actions) I can see that the transferred context size is reduced from 
140.3kB down to 23.55kB. The local difference is even more remarkable because I have packages, DLLs, and other obj content.


## Image size and SDKs

Container Structure Tests support so-called `commandTests`. They are the tests that can execute any CLI tool of your choice against your container and verify the produced output.

The syntax is pretty basic and somewhat limited. Tho we can still express our desires. 

We want our image to satisfy the following policies:

* No .NET Core SDK installed
* .NET Core runtime should be installed and registered
* No unit tests .dlls should be shipped

Let's turn these requirements into tests:

```yaml
commandTests:
    - name: ".NET Core runtime installed"
      command: "which"
      args: [dotnet]
      expectedOutput: ["/usr/bin/dotnet"]

    - name: "no SDK installed"
      command: "dotnet"
      args: ["--list-sdks"]
      excludedOutput: [".*/sdk]*"]
    
    - name: "no test dlls present"
      command: "find"
      args: ["/app", "-name", "*.Tests.dll"]
      excludedOutput: [".*.Tests.dll*"]
```

The first test executes `which` command and checks the standard output content with the provided string (this string is treated as a RegEx btw). This test will pass on the first run, so I can't call that a TDD approach, heh.

The second test runs the `dotnet --list-sdks` command and verifies that there are no matches in the output.

And the last one runs the `find` command which looks up for `*.Tests.dll` in the `/app` directory.

To fix the second test, we should use a [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) approach.

That will allow us to build and test our code in one container and then ship and run in another.

And the test dlls could be excluded by this [little fix](https://github.com/dotnet/sdk/blob/3704d0ae1e75166204f2ea154c37ca89097dc97d/src/Tasks/Microsoft.NET.Build.Tasks/targets/Microsoft.NET.Publish.targets#L63-L65) to the `.csproj` file.

```xml
  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <IsPublishable>false</IsPublishable>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
```

## Metadata tests

`metadataTests` can help us to ensure that the necessary environment variables are set, that our image has labels, the needed ports are exposed and the working directory is set correctly.


```yaml
metadataTest:
    env:
      - key: 'ASPNETCORE_URLS'
        value: 'http://+:80'
      - key: 'DOTNET_RUNNING_IN_CONTAINER'
        value: 'true'
    labels:
      - key: 'org.opencontainers.image.authors'
        value: 'https://github.com/asizikov'
      - key: 'org.opencontainers.image.source'
    workdir: "/app"
    exposedPorts: ["80"]
```

(I'm using label [naming schema](https://github.com/opencontainers/image-spec/blob/master/annotations.md#pre-defined-annotation-keys) by [Open Container Initiative](https://opencontainers.org/))

## Automate all the things!

As soon as we have a successful tests run locally 

![Green tests locally](/images/2020-09-container-tests/green-tests-local.png)

it's time to put things together and update our GitHub Actions workflow file.

I'm going to use the [Container Structure Test Action](https://github.com/marketplace/actions/container-structure-test-action) here.

which is extremely easy to use: provide it with a name of your image or a `tar` file with the image export results.

```yaml
    - name: run structure tests
      uses: plexsystems/container-structure-test-action@v0.1.0
      with:
        image: my-image:latest
        config: tests.yaml
```

The final action file would look like that: 

{%gist a8092cfaf780b5ff48549041c31134f9 %}

and the fixed Dockerfile is [here](https://github.com/asizikov/asp-net-container/blob/a1d35e57503479e68563f4c8bc1182f985b7028e/Dockerfile.fixed).

What could be better than a green CI pipeline?

![CI is green](/images/2020-09-container-tests/ci-green.png)

PS: while fixing our tests we also reduced the size of our image. Now its size is just 208Mb. A nice side benefit, eh?