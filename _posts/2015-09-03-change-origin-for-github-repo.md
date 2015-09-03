---
layout: post
title: Avoid typing user name when committing to GitHub repository
tags: github origin repository tip
---

Hi,

I use different accounts and different computers to work with GitHub repositories, so sometimes I face the situation when I don't have my SSH key generated for current environment.

I can still work with my command line tool , however I have to type my credentials every time I want execute the command against remote. [???]

[screen-shot1]

Actually I'm OK about typing the password, but not the user name. So what can I do (besides generating new SSH key and adding it to my Git/GitHub account) is to update the remove to have my user name in it. 

With an existing repository the flow will be the following:

{% highlight bash %}
git clone https://github.com/COMPANY/repository/

git origin update https://USER_NAME@github.com/COMPANY/repository/

{% endhighlight %}

Your user name, company/organisation repository name will be different.

Now the user name is not needed.
[screen-shot2]