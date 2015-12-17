---
layout: post
title: Fully automated Continuous Integration for your Open Source library for free
tags: github appveyor free
---

This is a long title. Well, the post is going to be long as well.

I want to show how you can build the CI pipeline using free services and tools.

* [GitHub](https://github.com) 
* [GitVersion](https://github.com/GitTools/GitVersion)
* [Appveyor](http://www.appveyor.com/) 

As an example I'm going to use my pet project: plugin for ReSharper. The reason is that the way you pack and publish R# extensions is slightly different from the regular NuGet package. I actually faced some limitations of NuGet.exe and Appveyor.

## GitHub 
Git and GitHub both are kind of industry standard for open source software development nowadays.  

I'm using slightly modified version of git flow for my project. I have two mandatory branches `master` and `develop`. 
`Master` branch contains released code marked with tags. `Develop` branch is for stable pre-release code. 
These two branches are configured [as protected][ProtectedBranches], so code can never be merged unless [CI server reports the successful build][ProtectedMerge]. This allows me to publish stable and pre release packages automatically.

The actual development is taking place in feature branches. The build started from feature branch will create an alpha package and publish it to separate NuGet feed provided by AppVeyor.

##GitVersion

I like the approach suggested by GitVersion. You can define package versions based on your branching model. 

The basic idea is simple: 

The build triggered by commit to Feature branch produces alpha package, beta packages come from develop branch, release candidates come from master. 
The tag produces stable version. 
If you want to get more backgrounds I recommend you a nice [couple][GVSemVer] of [posts][GVSemVer2] written by [my colleague](https://twitter.com/gusztavvargadr). 

##Appveyor 
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


[ProtectedBranches]: https://help.github.com/articles/about-protected-branches/
[ProtectedMerge]: https://github.com/blog/2051-protected-branches-and-required-status-checks
[GVSemVer]: http://gusztavvargadr.github.io/2015/09/20/1-everyday-gitflow-and-semantic-versioning/
[GVSemVer2]: http://gusztavvargadr.github.io/2015/10/30/2-gitversion-to-the-rescue/
[AppVeyorGitHub]: http://www.appveyor.com/docs