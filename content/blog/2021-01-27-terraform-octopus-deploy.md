---
date: "2021-01-27T00:00:00Z"
image: /images/2021/01-tf/octopus-to-the-cloud.jpg
title: Configure CD for Azure WebApp with Terraform Provider for Octopus Deploy
tags: 
 - devops
 - octopus deploy
 - terraform
slug: terraform-octopus-deploy 
---

In this post, I'm going to configure the continuous delivery process for Azure WebApp (Azure Function in this case, but that's pretty much the same) with Octopus Deploy. To make it a little bit interesting I'm going to use Configuration-as-Code approach with a brand new [Octopus provider for Terraform](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest).

![](/images/2021/01-tf/octopus-to-the-cloud.png)

Buckle up and let's get started...

## Tools

### Terraform

I'm going to need `terraform` on my machine:

![](/images/2021/01-tf/tf.png)

### Octopus Deploy

For this post, I've signed up for a [free tier of Octopus Deploy SaaS offering](https://octopus.com/start/cloud). Of course, the self-hosted version will work as well.

When signed up and configured my account I can generate a new API key that I'm going to use with Terraform

![](/images/2021/01-tf/api-key.png)

All configuration is going to be made against the default Space (`Spaces-1`), however, that's configurable too.

### Azure 

To connect Octopus Deploy with Azure I'm going to need a Service Principal.

```bash
# (!) make sure to follow the principle of least privilege here 
# and define the role and scope
az ad sp create-for-rbac --name sp-octopus-deploy
```

Grap the tenant Id, application Id, password, and subscription Id values. We'll heed them later.

### Azure Function

For the demo, I'm going to use this super simple Azure Function that replies to your GET request with a list of headers supplied.

The complete source code is [published here](https://github.com/asizikov/terraform-octopus-deploy/tree/main/src/echo_api).

the `.csx` gist is the following

{{< gist asizikov f06f44429255c87056eed034e3b589ef >}}

### Azure DevOps

I'm running the CI part in Azure DevOps.

Project configuration is pretty simple and it's terraform set up [can be found here](https://github.com/asizikov/terraform-octopus-deploy/tree/main/infra/azure_devops).

the pipeline itself it [here](https://github.com/asizikov/terraform-octopus-deploy/blob/main/build/pipeline.yaml).

Nothing complicated: 
 * Build Azure Function
 * Pack Azure DevOps artifact (zip)
 * Upload it to Octopus Deploy's built-in feed
 * Create new Octopus Deploy release
 * Trigger the deployment

 ![](/images/2021/01-tf/pipeline.png)

## Init

Terraform provider is no different from other providers and can be found on [Terraform Registry](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest)


{{< gist asizikov de5608266e39e4bd3656ee42971f4f9e >}}

and `terraform init` will download missing files

![](/images/2021/01-tf/init.png)

## Configuration

For the demo, I'm going to have this `terraform.tfvars` file to provide configuration values to Terraform.

{{< gist asizikov 527980e58759626fc4988833cc3a07e4 >}}


## Configure Environment

Now we're ready to start. At first, I'm going to configure a new [Environment](https://octopus.com/docs/infrastructure/environments).

> Environments are how you organize your deployment targets (whether on-premises servers or cloud services) into groups that represent the different stages of your deployment pipeline, for instance, development, test, and production.

{{< gist asizikov a36c0c2d6ef23f5aa412b7b7199a6f27 >}}

You'd probably have more than one, but that'll do for now.

## Deployment Target

The next crucial part is [Deployment Target](https://octopus.com/docs/infrastructure/deployment-targets)

>With Octopus Deploy, you can deploy software to Windows servers, Linux servers, Microsoft Azure, AWS, Kubernetes clusters, cloud regions, or an offline package drop. Regardless of where you're deploying your software, these machines and services are known as your deployment targets.

{{< gist asizikov f7c46cdc117a2f4fa8aa8709971ced5b >}}

I'm using `octopusdeploy_azure_web_app_deployment_target`, It's quite specific. I could have used a more generic [`octopusdeploy_deployment_target`](https://registry.terraform.io/providers/OctopusDeployLabs/octopusdeploy/latest/docs/resources/deployment_target) instead.

This is where I link Azure (my Service Principal) with Octopus.

Now we have a nice and clean infrastructure defined in Octopus.

![](/images/2021/01-tf/environment.png)

## Project

The next step is to configure our [Project](https://octopus.com/docs/projects).

> Projects let you create and manage your deployment processes, releases, and runbooks from the Octopus REST API and Octopus Web Portal.

{{< gist asizikov 3c38307ef53cff53631edad17ece29a2 >}}

I'm setting up an `Echo Api` project here, placing it into `Default Project Group` with the default [lifecycle](https://octopus.com/docs/releases/lifecycles).

![](/images/2021/01-tf/project.png)

## Deployment process 

And now the juicy part. 

{{< gist asizikov 46cf47377155fef292da378d0cb0fdca >}}

My deployment is not complicated actually, just one step that is picking up the package from a built-in feed and pushing it to my only environment. 

I wish all deployments were that simple...

![](/images/2021/01-tf/process.png)


## Release

Release creation process is driven by Azure DevOps pipeline. The process is split into two stages. `Release Creation` and `Deployment`.

### Create Release stage

{{< gist asizikov 2bbbcca82c0f05ad6ea42730c3b238b5 >}}

This stage uploads the build artifact to the built-in package feed, submits metadata and creates a new release.

![](/images/2021/01-tf/new-release.png)

### Deploy release stage

`OctopusDeployRelease` task will trigger a new deployment process

{{< gist asizikov 488aa1b1d3f9b6bd8dfd5a24c820e3dd >}}


![](/images/2021/01-tf/deployment.png)

Voilà, it's up and running.

![](/images/2021/01-tf/running.png)

## Import

The neat part about Octopus Deploy is its resource ID's system. It's always easy to find the id of the object which makes it super easy to import existing resources into your terraform state.

Let's assume that I already have an environment configured and I want to adopt it so that I can manage it with Terraform. 

```
https://cloud-eng.octopus.app/app#/Spaces-1/infrastructure/machines?environmentIds=Environments-7
```

The URL looks this way. `Environments-7` is the id of my environment. 

All it takes to import the environment is to declare the resource and type one command:


{{< gist asizikov 2301838944f3e1d5f4259451fa9a2a13 >}}

this is such a breath of fresh air comparing to long and not so easy way to get Ids in Azure :)

Happy provisioning!