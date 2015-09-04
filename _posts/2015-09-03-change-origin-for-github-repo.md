---
layout: post
title: Avoid typing user name when committing to GitHub repository
tags: github origin repository tip
---

Hi,

I use different accounts and different computers to work with GitHub repositories, so sometimes I face the situation when I don't have my SSH key generated for current environment.

I can still work with my command line tool, however I have to type my credentials every time I want to push or pull to the remote.

[screen-shot1]

Actually I'm OK about typing the password, but not the user name. So what can I do (besides generating new SSH key and adding it to my Git/GitHub account) is to update the remove to have my user name in it. 

First of all let's check what is the value of our origin url.
{% highlight bash %}
git config remote.origin.url
{% endhighlight %}

We'll get something like that:

{% highlight bash %}
https://github.com/COMPANY/repository.git
{% endhighlight %}

Now we can update the origin url.

{% highlight bash %}
git config remote.origin.url https://USER_NAME@github.com/COMPANY/repository/
{% endhighlight %}

Your user name, company/organization repository name will be different.

Now the user name is not needed.
[screen-shot2]