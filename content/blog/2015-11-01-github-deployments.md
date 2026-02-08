---
date: "2015-11-01T00:00:00Z"
tags: 
    - github 
    - devops
title: GitHub Deployment statuses
slug: github-deployments
---


It’s very important to collect and track as much information as you can about your system. We have logging, monitoring, reports and analytics. All the systems that we build are not just packages which are deployed to the server/computer or device. Everything starts with an Issue Tracking system and, through the code, goes to production. The code and the process of coding both look like an important part of the system, and it makes a lot of sense to collect and store all the data about code. 

The actual process of programming is tracked by VCS. The majority of teams can easily tell you where this line of code came from. Author, branch name, and often the issue identifier are stored in association with the commit. If you don’t modify the history, then you’ll be able to track down the way the developer was building the feature. 

GitHub provides you with an extra level of storing information about your code. It’s definitely not just the web UI for your repository. The Pull Request collects all the commits related to the feature, it tracks the discussion and as long as it’s a part of your history, you cannot [delete the PR](https://stackoverflow.com/questions/18318097/delete-a-closed-pull-request-from-github). That makes it a great place to aggregate other events.

The feature, which is well known is the [GitHub Statuses API](https://stackoverflow.com/questions/18318097/delete-a-closed-pull-request-from-github).

![Statuses in action](/images/github-deployments/statuses.png)

Statuses are most often sent by CI server. However CI is not the only possible source of this information. Anyway, statuses are linked to the commit and you can use them in your process. For example, with the combination of [protected branches](https://help.github.com/articles/about-protected-branches/) and statuses you can prevent you from merging code that cannot be built into master branch.

The other feature of GitHub, which I discovered not so long ago, is Deployment Statuses. It’s quite simple at the moment. The only thing it does is link the branch/commit or tag with the event of deployment and sends notifications around.

![Deployment created](/images/github-deployments/deployment.png)

Successful deployment on the test environment is just another confirmation for your teammates that your changes are fine and it’s time to merge your code.

![Deployments in action](/images/github-deployments/deployments.png)

That’s how it looks like. Well, except the fact that I don’t usually talk to myself.

Requesting a new deployment using Octokit.net is relatively simple:


{{< gist asizikov 25715ab8fb008f9a3e97 >}}

Updating the status is not complicated as well:

{{< gist asizikov cd70f64ec3659f095074 >}}

As I mentioned earlier, the deployment can be requested for a particular commit, for the branch or for the tag. 

# Links

* [Statuses API](https://developer.github.com/v3/repos/statuses/)
* [Deployments API](https://developer.github.com/v3/repos/deployments/)
* [Deploying branches to GitHub](http://githubengineering.com/deploying-branches-to-github-com/)


