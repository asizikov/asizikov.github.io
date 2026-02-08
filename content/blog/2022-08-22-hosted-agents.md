---
date: "2022-08-22T00:00:00Z"
title: AzureDevOps and hosted agents
tags: 
 - devops
 - azure-devops
 - hosted-agents
slug: ado-hosted-agents
---

## TLDR

Today I'm going to talk about self-hosted build agents.

After reading this post you will be able to host your own Windows or macOS build agent (we'll run it on your laptop) and use it to execute AzureDevOps Pipeline.

## Motivation

It's common to run different steps of your build and deployment on different machines. You build steps can have different requirements, for example:

* Compute and disk space requirements
* Software requirements (SDKs, OSs)
* Lifecycles
* Location (geolocation, network isolation)

AzureDevOps and GitHub support two different types of build agents:

* GitHub Hosted (Microsoft Hosted)
* Selfhosted

From now on I'm going to talk about Azure DevOps, but the same patterns and ideas apply to any other modern build systems.

## Microsoft-Hosted Build Agents

Microsoft-hosted agents are convenient, because this is probably the simplest way to execute our build pipelines.

Managed agents have these advantages:

* maintenance and upgrades are taken care of for you.
* every pipeline run gets a fresh VM (it's discarded after use)
* consumption based billing

The agent choice is generic and broad enough to cover most of the common use cases.

![](/images/2022-08-hosted-agents/tools.png)

### Hardware

Microsoft-hosted agents that run Windows and Linux images are provisioned on Azure general purpose virtual machines [Standard_DS2_v2](https://docs.microsoft.com/en-us/azure/virtual-machines/dv2-dsv2-series#dsv2-series). These virtual machines are co-located in the same geography as your Azure DevOps organization.

Agents that run macOS images are provisioned on Mac Pros. These agents always run in US and Europe irrespective of the location of your Azure DevOps organization.

All of these machines have 10 GB of free disk space available for your pipelines to run.

### Security

* Microsoft-hosted agents run on Azure public network, they are not assigned public IP addresses. That means external entities cannot target Microsoft-hosted agents.
* You don't have to worry about security issues when running code, frameworks or containers downloaded from Internet.

### limitations

However it's important to understand the limitations

* Diskspace is limited
* No way to RDP or SSH to the build agent and troubleshoot the build
* No way to upgrade the hardware
* macOS agents run in US and Europe only, so you may have issues with data sovereignty 
* No way to join machines directly to your corporate network
* No way to pre-install custom software (other than through tool installer tasks)
* No control over software upgrades, cannot use legacy SDKs etc
* No control over the waiting time (it depends on the load)

To address these issues Microsoft (and other vendors) allow us to use self-hosted agents.

## Self-hosted agents

Self-hosted agents are the agents which you set up and manage on your own.

### Types

* macOS agent
* Linux agent (x64, ARM, RHEL6)
* Windows agent (x64, x86)
* Docker agent

### Overview

![](/images/2022-08-hosted-agents/firewall.png)

Managed agents will run in a cloud, and won't have access to your private network, however self-hosted agents can be deployed into your VNet, behind the corporate firewall.

## Self-Hosted agent

Since I'm running Windows right now, I'll show you how to set up a Windows agent.

There are several steps which I need to complete in order to set up a build agent:

## Obtain a personal access token  (PAT)

![Go to your security details.](https://docs.microsoft.com/en-us/azure/devops/repos/git/media/select-personal-access-tokens.jpg)

and create a PAT with these scopes

![](/images/2022-08-hosted-agents/pat.png)

create a new pool

![](/images/2022-08-hosted-agents/new-pool.png)

create a new agent

![](/images/2022-08-hosted-agents/new-agent.png)

Download the agent and configure it

```powershell
PS C:\> mkdir agent ; cd agent
PS C:\agent> Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-2.179.0.zip", "$PWD")
```

![](/images/2022-08-hosted-agents/configure.png)

If everything is good, you should be able to see your new agent: 

![](/images/2022-08-hosted-agents/running.png)

## Verify

We can create simple pipeline

![](/images/2022-08-hosted-agents/build.png)

And verify if it's running on my PC

![](/images/2022-08-hosted-agents/output.png)

As you can see we have our pipeline running on my computer. This means I can use my own hardware such as GPU or devices connected to my laptop. Also I have access to internal network, printers and whatnot.

In reality you would probably run the self-hosted agent on VM or a dedicated physical machine.

## Important notes

It is a **bad practice to configure your self-hosted agents for public repositories**. Keep in mind that anyone can create a fork of your repository, modify a pipeline and execute code on your machine. It's ok for a test, it's ok to configure it for private repo, but please be careful, and don't forget to shut down the agent when you don't need it anymore.