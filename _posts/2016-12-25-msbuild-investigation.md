---
layout: post
title: Write code for the reviewer, not for the compiler 
image: /images/code-for-reviewer/matrix.png
---

Recently I took latest changes from git, merged `dev` to my current branch and descided to run few integration tests. You know, just to be sure.

What do I see? well... this.

That doesn't sound right, though it's typically easy to fix. 

Thanks God we have MSBuildStructuredLog tool. 
https://github.com/KirillOsenkov/MSBuildStructuredLog

Let's take a look at 
