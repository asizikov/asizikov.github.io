---
date: "2015-09-05T00:00:00Z"
tags: 
    - github 
    - tools
title: Avoid typing user name when committing to GitHub repository
slug: change-origin-for-github-repo
---

I use different accounts and different computers to work with GitHub repositories, so sometimes I face the situation when I don't have my SSH key generated for the current environment.

I can still work with my command line tool, however I have to type credentials every time I want to pull or push to the remote.

![credentials required](/images/github-origin/password-required.png)

Actually I'm fine with typing the password, but not the user name. So what can I do (besides generating new SSH key and adding it to my Git/GitHub account) is to update the remote to have my user name in it. 

First of all let's check what is the value of origin url.
{{< highlight bash >}}
git config remote.origin.url
{{< / highlight >}}

We'll get something like that:

{{< highlight bash >}}
https://github.com/OWNER/repo.git
{{< / highlight >}}

Now we can update the origin url.

{{< highlight bash >}}
git config remote.origin.url https://USER@github.com/OWNER/repo.git
{{< / highlight >}}

Your user name, repository owner and repository name will be different.

And the user name is not needed any more.

![credentials not required](/images/github-origin/no-password.png)