---
date: "2022-11-26T00:00:00Z"
title: GitHub Environment Protection with Tags
tags: 
 - github-actions
 - github
 - devops
slug: environment-protection-tags
---

A recent request from a customer caught my attention. They were trying to set up a CD pipeline that should deploy packages to production when there is a new tag pushed to the repository. At the same time, they wanted some control over the process so that not everyone can init a release by pushing a tag to the repository.

In a more standard scenario, you would use a branch protection rule to prevent pushing to `main` branch, together with environment protection rule that prevents deployments to production from any other branch but `main`. However, in this case, they are using tags to trigger the deployment. So, we need to protect the tags. Currently, it's not obvious that you can restring environments to specific tags or tag patterns. Let's see how we can do that.

## Protecting tags

The first thing we need to do is to create a new tag protection rule. The rule should look like this:

![Tag Protection Rule](/images/2022-11-env-tags/tag-protection-rule.png)

The rule is pretty simple. It only allows the users with admin or maintain permissions in the repository to be able to create tags. This is exactly what we need.

## Protecting environments

Now, we need to set up new environment protection rule. We will call it `production`. The rule should look like this:

![Environment Protection Rule](/images/2022-11-env-tags/env-protection-rule.png)

Note that I'm using the tag name pattern to match the tags that should be deployed to production, despite the UI saying "branches". The pattern is `release_*` This means that any tag that starts with `release_` is allowed to be deployed to production.

## Deployment workflow

And finally, I'm creating a simple workflow that will deploy the package to production when a new tag is pushed to the repository. The workflow looks like this:

![Workflow](/images/2022-11-env-tags/workflow.png)
[https://gist.github.com/asizikov/89d1b663593d9b934622d51e27658287](https://gist.github.com/asizikov/89d1b663593d9b934622d51e27658287)

## Protection rules in action

Let's see what it looks like when an unprivileged collaborator tries to push a tag to the repository.

![Push prevented](/images/2022-11-env-tags/push-prevented.png)

Also, an unprivileged collaborator cannot create a new release with protected tag.

![Release prevented](/images/2022-11-env-tags/release-not-created.png)

So far so good. Now let's see what happens when maintainer pushes a tag to the repository.

![Push allowed](/images/2022-11-env-tags/push-goes-through.png)

And the workflow is triggered and completed successfully.

![Workflow completed](/images/2022-11-env-tags/deployment-workflow.png)

The same applies to creating a new release with a protected tag.

![Release created](/images/2022-11-env-tags/release-created.png)


## Option

Another option is to trigger the workflow on a new release:

```yaml
# trigger workflow on release with tag name pattern
name: ⚙️ deployment on release created
on: 
  release:
    types: [published]
jobs:
  deployment:
    if: github.event.release.tag_name == 'release_*'
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: ▶️ Run a one-line script
        run: echo 'deployment to production'
```

This will trigger the workflow when a new release is created with a tag that matches the pattern. The downside is that you need to create a new release every time you want to deploy to production. Also, you may need to add extra scripting to the workflow to make sure that the tag is created in `main` branch.

```bash
git branch --contains tags/release_011
```

## Conclusion

Would I recommend using this approach? I would say it depends on your use case. The problem with it is that you cannot easily prevent users from tagging a commit that is not in the `main` branch, the same applies to releases, the UI allows you to pick any branch.

When your situation doesn't allow you to deploy on every commit/merge to `main` branch you may want to consider other branching strategies, like GitFlow, where you have a dedicated branch for releases. In this case, you can use branch protection rules to prevent pushing to `main` and `release` branches.

### Useful links

* [Configuring tag protection rules](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/managing-repository-settings/configuring-tag-protection-rules)
* [Using environments for deployment](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment)
* [Demo repository](https://github.com/asizikov/environment-protection-tag)

### Disclosure

> I'm employed by GitHub at the time of writing this post. All opinions are my own.