---
layout: post
title: Fully automated Continuous Integration for your Open Source library for free
tags: github appveyor free
---

![open source is commumism](/images/ci-for-free/communism.png)
This is a long title. Well, the post is going to be long as well.

I want to show how you can set up the CI pipeline using free services and tools.

* [GitHub](https://github.com) 
* [GitVersion](https://github.com/GitTools/GitVersion)
* [AppVeyor](http://www.appveyor.com/) 

As an example I'm going to use my pet project: [AsyncSuffix][AsyncSuffixGitHub] plugin for ReSharper. The reason is that the way you pack and publish R# extensions is slightly different from the regular NuGet package. I actually faced some limitations of NuGet.exe and AppVeyor.

## GitHub 
Git and GitHub both are kind of industry standard for open source software development nowadays.  

I'm using slightly modified version of git flow for my project. I have two mandatory branches `master` and `develop`. 
`Master` branch contains released code marked with tags. `Develop` branch is for stable pre-release code. 
These two branches are configured [as protected][ProtectedBranches], so code can never be merged unless [CI server reports the successful build][ProtectedMerge]. This allows me to publish stable and pre release packages automatically.

The development is taking place in feature branches. The build triggered from the feature branch will create an alpha package and publish it to separate NuGet feed provided by AppVeyor.

## GitVersion

I like the approach suggested by GitVersion. You can define package version based on your branching model. 

The basic idea is simple: 

* The build triggered by commit to Feature branch produces alpha package, beta packages come from develop branch, release candidates come from master. 
* The tag produces stable version. 

If you want to get more backgrounds I recommend you a nice [couple][GVSemVer] of [posts][GVSemVer2] written by [my colleague](https://twitter.com/gusztavvargadr). 

## AppVeyor 
I'm using AppVeyor as a CI server.

AppVeyor is a free cloud build server which is [easy to integrate][AppVeyorGitHub] with your GitHub repository.
Creating the account is simple, you can log in using your GitHub and you’re done. 

There are two ways to configure build for the project. 
* Using UI 
* By placing [AppVeyor.yml](http://www.appveyor.com/docs/appveyor-yml) file to the root of the repository. 

First option is good for testing, but committing configuration file to the repository gives you ability to track versions. 

## The goal

As I already mentioned, I work in a feature branch and I want to make sure that build is not broken. 
That means that build server constantly compiles and packs code. 
However code in a feature branch is most likely unstable and it's better not to publish it to the official feed.

On the other hand when the feature is complete, tested, and merged to `develop` branch I'm more than happy to publish prerelease package.
I'm dogfooding anyway.

External pull requests are different story. The build process must be triggered (how else can I be sure that it's safe to accept it?). But I don't want to have any packages created.

Let's summarise it:

* Build process is triggered by any commit, merge or tag action
* Build process depends on the branch name

## Configuration

First of all we need to agree on the branch naming. Typically it depends on the branching model you use.
For GitFlow I'm using the following naming convention: 

* `master` and `develop` - for stable code
* `feature/*` - for unfinished features

My `AppVeyor.yml` will look like: 

{% gist 0498fdbd14888bea9b17 %}

Unfortunately there is no way to have common sections in the config file (well, at least at the moment), 
so we have no other option but having very similar configurations. However AppVeyor evolves very quickly and we can expect some improvements.

Once we have templates defined we can start with actual build steps.

Install GitVersion from [Chocolatey][ChocolateyOrg]

```
install:
    - choco install gitversion.portable -y
```

Define environment variables (different for each branch configuration)

```
environment:
    resharper_nuget_api_key:
      secure: RjiHK3Oxp74LUrI1/vmc2S36zOSRLxFM1Eq0Qn4hixWiou11jFqUbW2ukMNXrazP
```

Important note: the api key is [encrypted](https://ci.appveyor.com/tools/encrypt) and can be published to the public repository.

Restore NuGet packages and set the build version.

```
before_build:
    - ps: nuget restore
    - ps: gitversion /l console /output buildserver /updateassemblyinfo /b (get-item env:APPVEYOR_REPO_BRANCH).Value
```

The build itself:

```
build:
    project: AsyncSuffix.sln
```

Now it's time to create NuGet package. 
AppVeyor has a feature to create packages automatically: 

![CreatePackage](/images/ci-for-free/package-projects.png)

Doesn't work for me though. At the moment it [ignores the content](http://help.appveyor.com/discussions/problems/2698-when-using-publish_nuget-true-my-nuspec-is-ignored-and-it-is-packaging-the-csproj-instead) of `.nuspec` file, 
and performs `nuget pack` command on the `csproj` file. 

That's why I have to define an `after_build` step:

```
after_build:
    - ps: nuget pack AsyncSuffix/AsyncSuffix.nuspec -Version (get-item env:GitVersion_InformationalVersion).Value
```    

I have to specify the version, because the `nuget pack` command reads package version from assembly annotations. 
Unfortunately I have this annotation:

{% gist 4f9eb8f0375e3ce81de0 %}

As you can guess `RegisterConfigurableSeverityAnnotation` is declared in one of R# SDK's assemblies. NuGet fails to load it and falls back to `1.0.0` version.

The final step is different for every branch configuration:

```
- ps: if(-not $env:APPVEYOR_PULL_REQUEST_NUMBER){ 
        nuget push *.nupkg 
            -ApiKey (get-item env:resharper_nuget_api_key).Value 
            -Source https://resharper-plugins.jetbrains.com 
      }
```

If `APPVEYOR_PULL_REQUEST_NUMBER` environment variable is defined that means that we're currently performing a synthetic build of merge commit from Pull Request.
The `ApiKey` will be transparently decrypted and the package will be published:

![Success](/images/ci-for-free/pushing-package.png)


P.S. You can definitely do more than publishing packages. You can [execute tests](https://www.appveyor.com/docs/running-tests), custom build scripts (PowerShell, [FAKE](http://fsharp.github.io/FAKE/)) or even [deploy](https://www.appveyor.com/docs/deployment) your application.

[ProtectedBranches]: https://help.github.com/articles/about-protected-branches/
[ProtectedMerge]: https://github.com/blog/2051-protected-branches-and-required-status-checks
[GVSemVer]: http://gusztavvargadr.github.io/2015/09/20/1-everyday-gitflow-and-semantic-versioning/
[GVSemVer2]: http://gusztavvargadr.github.io/2015/10/30/2-gitversion-to-the-rescue/
[AppVeyorGitHub]: http://www.appveyor.com/docs
[AsyncSuffixGitHub]: https://github.com/asizikov/AsyncSuffix
[ChocolateyOrg]: https://chocolatey.org/
[ReSharperDev]: https://www.jetbrains.com/resharper/devguide/Extensions/Packaging.html
