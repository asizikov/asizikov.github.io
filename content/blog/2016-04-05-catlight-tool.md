---
date: "2016-04-05T00:00:00Z"
tags: 
    - tools
    - appveyor
    - devops
title: CatLight
slug: catlight-tool
---

It's hard to imagine modern development without continuous intergation and unit-tests.

At work I hardly pay attention on the process, it just works: I push code 
to GitHub, later on TeamCity picks up changes and starts the build, 
and few minutes after I receive Slack or email notification about the result.
However for my personal projects things are different. I'm using free plan on AppVeyor. 
It works pretty well except the fact that your build might stay in the queue for a while.

Last week I found a nice application called [CatLight](https://catlight.io/). 
That's a tiny tool wich shows you build notifications in a tray. Ok, not that tiny as it's built on electron and .NET :) 
Works for both Windows and OSX.

Setup is very straightforward: provide your AppVeyor API key

![screenshot](/images/catlight-tool/api-key.png)

Select all the projects you want to be notified about

![screenshot](/images/catlight-tool/select-projects.png)

Select all the events you want to be aware of

![screenshot](/images/catlight-tool/settings.png)

Trigger the build

![screenshot](/images/catlight-tool/build-started.png)

and wait for the green light.

![screenshot](/images/catlight-tool/build-finished.png)







 