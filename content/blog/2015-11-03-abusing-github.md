---
date: "2015-11-03T00:00:00Z"
tags: 
    - github 
    - tools
title: How to convince Linus Torvalds to contribute to your project
slug: abusing-github
---

Look at all those famous people committing to some random developer’s [boring repository](https://github.com/asizikov/gang/graphs/contributors). 

![gang](/images/github-author/gang.png)

Why would they do that? In fact, they don’t. 

In general, git is just a tool that allows you to create [patches](https://git-scm.com/docs/git-format-patch) and distribute them around [by email](https://git-scm.com/docs/git-send-email).

When you create a commit, it will be signed with your name and email. Look at the author part here:

![commiter](/images/github-author/committer.png)

You have your name listed twice for every commit. You are both the author and the committer. Technically, the author is the one who created the patch, and the committer is the person who applied the patch.

By default, both values are taken from your [git](https://help.github.com/articles/setting-your-username-in-git/) [config](https://help.github.com/articles/setting-your-email-in-git/)

However, there is a way to override them:

{{< gist asizikov 3e9d0ca4ee9d0b4d60ce >}}

This command will write any name and email to both, the committer and the author fields.

# Why would you do that?

The real life use case scenario: imagine you have to work with a repository on someone else's computer. Or you work from you personal computer, but you want to commit your changes to your work repository. You might want to use the proper user name to keep the history clean and correct.

# Disclaimer

First of all, this is not a security issue. You don't actually gain access to the person’s git account and repositories. 
The only thing that GitHub is doing here is linking the fake commit to their account based on their e-mail address. This activity will not be shown on their profile page.

So use it wisely and do not abuse it too much :)

