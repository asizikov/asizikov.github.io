---
date: "2016-06-12T00:00:00Z"
image: /images/dev-tools-2016-win/old-tools-8.png
title: Dev environment 2016. Windows.
tags:
    - tools
slug: dev-environment-2016-win
---

![tool](/images/dev-tools-2016-win/old-tools-8.png)

I've changed a job last month and had to build up my dev environment from scratch again. While doing that I decided to write down some thoughts about it. 

I guess it might be interesting to look back at some point and see how does it evolve.

## Background

At my previous employer, we were very into Virtual Machines. We had different base VMs which every developer can download. 

That's extremely handy when a new hire has nothing to do, but to install just a couple of tools that are not standard, and enter some credentials.

You remember those dialogs: 

>--- I can't build the project. Packages are missing...
>
--- Oh, yep, you need to add this private NuGet feed.
>
>...
>
>--- I can't run the project locally. 
>
>--- Yeah, I think you need so put these lines into your `hosts` file.

Well, now it's all gone.

It's not only limited to this scenario. You want to experiment with a new unstable version of the framework (yes, [.NET Core RC-final-almost-stable](https://twitter.com/NETCoreReadyYet/status/735972346436161536), I'm talking about you) and you don't want to mess up with your dev machine?

Just fire a new VM up. 

Got a neat idea for a hackathon, but you think that JDK is not what you need on your computer? Giving a tech demo on the local meetup?  

A VM comes to the rescue.

Got a new computer? Just copy the VM over and you're up to speed in 20 minutes.

## Back to the topic

So, what do I have on my base VM? 

### Frameworks

I'm a .NET web developer, so nothing special here:

* .NET framework 4.5.2 and 4.6
* Node.js (npm, gulp)

### IDE and editors

![vscode](/images/dev-tools-2016-win/vscode.png)

* VS 2015 with a set of extensions
    * [NoGit](https://visualstudiogallery.msdn.microsoft.com/146b404a-3c91-46ff-932a-fb0f8b826f94).
    * [PVS Studio](http://www.viva64.com/en/). A static analysis tool for C#.
    * [ReSharper](https://www.jetbrains.com/resharper/). A developer productiviy tool.
    * [OzCode](http://www.oz-code.com/). A debugging addon for VS. Makes my life much easier.
    * [SpecFlow](https://visualstudiogallery.msdn.microsoft.com/c74211e7-cb6e-4dfa-855d-df0ad4a37dd6). A port of Cucumber.  
* ReSharper goes with a handfull of [extensions](https://resharper-plugins.jetbrains.com) as well
    * [ConfigureAwaitChecker](https://resharper-plugins.jetbrains.com/packages/ConfigureAwaitChecker.v9/)
    * [ReSpeller](https://resharper-plugins.jetbrains.com/packages/EtherealCode.ReSpeller/)
    * [Mnemonics Live Templates](https://resharper-plugins.jetbrains.com/packages/JetBrains.Mnemonics/)
    * [Enhanced Tooltip](https://resharper-plugins.jetbrains.com/packages/JLebosquain.EnhancedTooltip/)
    * [Heap Allocations Viewer](https://resharper-plugins.jetbrains.com/packages/ReSharper.HeapView.R2016.1/)
    * [Shaolinq.ReSharper](https://resharper-plugins.jetbrains.com/packages/Shaolinq.ReSharper/)
    * [AsyncSuffix](https://resharper-plugins.jetbrains.com/packages/Sizikov.AsyncSuffix)
    * [Mr. Eric](https://resharper-plugins.jetbrains.com/packages/Sizikov.MrEric/)
* [Rider](https://www.jetbrains.com/rider/). A cross-platform C# IDE based on the IntelliJ platform and ReSharper. 
* [Visual Studio Code](http://code.visualstudio.com/) with extensions
    * C#
    * Markdown preview
    * Spelling and Grammar checker (being the only non-native speaker in a team is hard :) )

### File Managers and command line shell

![choco](/images/dev-tools-2016-win/choco.png)

* [Far manager](http://www.farmanager.com/index.php?l=en). An old school two panels file manager.
* [ConEmu](https://conemu.github.io/). A powerful and advanced console window.
* [Chocolatey package manager](https://chocolatey.org/). A package manager for Windows.

### Source control

![GitKraken](/images/dev-tools-2016-win/gitkraken.png)

* [GitBash](https://git-scm.com/downloads)
* [GitKraken](https://www.gitkraken.com/) A cross-platform UI tool for Git.

GitKraken is quite heavy and not super fast as most of the electron.js based tools are, but I find it's history tree view very readable. The merge tool is not bad at all. 
 
I do most of the git related operations in git bash, though.

## Debugging and profiling

![linqpad](/images/dev-tools-2016-win/linqpad.png)

* [DotPeek](https://www.jetbrains.com/decompiler/). A free .NET decompiler.
* [DotMemory](https://www.jetbrains.com/dotmemory) .NET memory profiler.
* [WinDbg](http://www.windbg.org/).
* [Fiddler](http://www.telerik.com/fiddler). A free web debugging proxy.
* REST and Http clients. I use two, can't decide which one I prefer over an other.
    * [ARC](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo)
    * [Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop)
* [LINQPad](https://www.linqpad.net/) A .NET programmer’s playground.

### Communication

* Slack
* Skype

### Other tools
* [FoxitReader](https://www.foxitsoftware.com/products/pdf-reader/). A PDF reader.
* [ScreenToGif](https://screentogif.codeplex.com/). 
* [Paint.NET](http://www.getpaint.net/index.html). A free image and photo editing software. 
* IIS and SQL Express 

### Web tools and services

Besides all the tools above which I have installed locally there are web services I use on a pretty much daily basis.

* [requestb.in](http://requestb.in/). An easy to use HTTP request inspector.
* [AppVeyor](https://www.appveyor.com/). A free CI/CD service for my open source projects.
* [regex101.com](https://regex101.com/). A super awesome regular expressions builder and debugger.
* [Toggl](https://toggl.com). A time tracker.


## Batch install

Most of the tools could be installed from Chocolatey [gallery](https://chocolatey.org/packages).

```bash
choco install dotnet4.5.2 linqpad -y
```

I prefer to have all the tools grouped into `.config` files: 

```xml
<!-- commandline-tools.config -->
<?xml version="1.0" encoding="utf-8"?>
<packages>
  <package id="git" />
  <package id="far" />
  <package id="nuget.commandline" />
  <package id="conemu" />
</packages>
```

and they could be installed all together.

![choco-install](/images/dev-tools-2016-win/choco-install-config.png)

## A call to action

Are there any tools around which are worth to try? 

Please share in comments. I'm always keen on trying new things.
