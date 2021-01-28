---
date: "2016-12-24T00:00:00Z"
image: /images/2016-12-msbuild-investigation/msbuild-log-dll-resolved-overview.png
title: Investigating unexpected MSBuild behavior
tags:
    - tools
slug: msbuild-investigation
---

Recently I took latest changes from git, merged `dev` to my current branch and decided to run few integration tests. You know, just to be sure.

What do I see? well... this.

>Could not load file or assembly '`Microsoft.Data.Edm, Version=5.6.4.0, Culture=neutral, PublicKeyToken=...` or one of its dependencies.

That doesn't sound right, though it's typically easy to fix. 

The first reaction is to consolidate NuGet packages in my Solution. All the installed packages have same version `5.7.0` though.

![Package.config sample](/images/2016-12-msbuild-investigation/installed-package-version.png)

Well, that happens when a merge conflict was poorly resolved. Perhaps one of the project files has a reference to an old version. 

Full-text search shows that each `.csproj` file has correct path in the `<Reference>` tag. 

Curiouser and curiouser!

dotPeek confirmed that the `Microsoft.Data.Edm.dll` version in the `\bin` folder is indeed incorrect. That's odd because I don't have an old version anywhere close to my solution. So where does it come from?

Reading raw MSBuild logs is some kind of an art, so this is where [MSBuildStructuredLog](https://github.com/KirillOsenkov/MSBuildStructuredLog) comes to the rescue. 

Make sure to add it to your toolbelt! This is a very handy tool for MSBuild logs analysis.

Let's run a verbose build and look at the log. The log is very detailed, however, I think that DoubleWrites section a good starting point.

And here we go: `Microsoft.Data.Edm` is initially taken from local `packages` folder and after that suddenly overwritten by the wrong version from my `.NETFramework` folder.

![MSBuild log](/images/2016-12-msbuild-investigation/msbuild-log-double-writes.png)

Still not clear why.

Luckily we can search through the logs. We're interested in resolve messages here: 

`Resolved file path is "C:\Program Files (x86)\Microsoft WCF Data Services\`

![Search](/images/2016-12-msbuild-investigation/msbuild-log-dll-resolved-overview.png)

This is our "Ahaaa" moment! As you can see on a screenshot above while building project 1 (see the red number, eh?) ResolveAssemblyReferences target picks up a wrong version of that DLL which is required by project 2.

Project 1 has a reference to project 2, and indeed has no references to `Microsoft.Data.Edm` dll. 

![VS Project References](/images/2016-12-msbuild-investigation/project-references.png)

And now it's somewhat clear what's going on. On the screenshot below we can see a part of the build log, where unimportant log entries were omitted. First occurrences of project 1 and project 2 are marked on the screenshot.

![MSbuild log. Overview](/images/2016-12-msbuild-investigation/msbuild-log-build-order.png)

It seems that MSBuild failed to resolve a path to `Microsoft.Data.Edm` while preparing project 2, and that's why an old version was used for project 1. However, project 2 has all the references configured correctly. 
Once the incorrect DLL version was resolved it's being used for all remain projects.

I'm not sure if that's an expected behavior. To be honest I'm a bit confused here. At least as a temporary solution I can add `Edm` package to project 1. That will fix my problem for now.
 
