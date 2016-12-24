---
layout: post
title: Write code for the reviewer, not for the compiler 
image: /images/code-for-reviewer/matrix.png
---

Recently I took latest changes from git, merged `dev` to my current branch and descided to run few integration tests. You know, just to be sure.

What do I see? well... this.

>Couldn't load Assembly `Microsoft.Data.Edm` version 5.6.4.0 or one of it's dependencies.

That doesn't sound right, though it's typically easy to fix. 

This nuget package has no dependencies, so it's clear that this particular library has a wrong version.

First reaction is to consolidate NuGet packages in my Solution. All the installed packages have same version `5.7.0` though.

![Package.config sample](/images/2016-12-msbuild-investigation/installed-package-version.png)

Well, that happens when a merge conflict was poorly resolved. Perhaps one of the project files has a reference to 
and old version. 

Full text search shows that each `.csproj` file has correct path in the `<Reference>` tag. 

Ok, that's getting more and more interesting, isn't it? 

dotPeek confirmed that the `Microsoft.Data.Edm.dll` version in the `\bin` folder is indeed incorrect. So where does it come from? 

Luckely we have [MSBuildStructuredLog](https://github.com/KirillOsenkov/MSBuildStructuredLog) tool. This is a greate tool to analyze MSBuild logs.

Let's run a verbose build and look at the log. The log is very detailed, however I think that Double writes section a good starting point.

And here we go: `Microsoft.Data.Edm` is suddenly taken from my `Program Files` folder.

![MSBuild log](/images/2016-12-msbuild-investigation/msbuild-log-double-writes.png)

Still not clear why.

`MSBuildStructuredLog` has a search option. Let's look `WCF Data Services` we'll find this:

![MSBuild log](/images/2016-12-msbuild-investigation/msbuild-log-search.png)

Ahaa MSBuild was trying to resolve a dependency, but didn't find a local reference and had to fall back to `Framework` folder.

Ok.





Let's take a look at 
