---
date: "2023-01-25T00:00:00Z"
title: Github Container Registry. Keeping traffic below the free tier limit.
tags: 
 - github-actions
 - azure
 - devops
slug: ghc-acr-traffic
---

Github Container Registry is a great service. It's free for public repositories and it's integrated with Github Actions. However, it's not free for private repositories. The free tier allows 1000 GB of traffic per month. If you exceed the limit, you'll be charged per GB. The up-to-date pricing can be found [here](https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages#about-billing-for-github-packages).

Paid and enterprise customers will have different limits and prices. However it's still a good idea to be cautious of the traffic limits and be able to save some money.

I had a conversation with GitHub user on Mastodon (I know, I know) and he was confused about the traffic limits and the logic behid them. I decided to write a post about it.

## How does the traffic limit work?

The traffic limit is calculated based on the number of pulls. The limit is 1000 GB per month. However if you pull an image while authorised via your `GIT_HUB_TOKEN`, the traffic is not counted. So in other words pulls that you make as a part of your CI/CD pipeline (GitHub Actions only) are not counted.

A typical use case would be the pull of custom base images, or an image that you use to execute sertain tasks as a part of your pipeline.

However, if you pull an image from outside of GitHub (e.g. from your Kubernetes cluster), the traffic is counted.

At the same if you use an external registry (e.g. Azure Container Registry) your cluster will be able to pull images without any traffic limits. Of course if your cluster is located in the same region as your ACR. But at the same time if you pull images as a part of your CI/CD pipeline, you'll be charged for eggress traffic.

So, that seems that in either way you'll be charged. But there is a way to avoid it.

## A sample app architecture

To have a concreet example I'm going to use a sample architecture of your application, its infrastructure and CI/CD pipeline.

![Architecture](/images/ghc-acr-traffic/architecture.png)

As you can see, we have a Managed Kubernetes cluster in Azure. We keep our code in GitHub, and we use GitHub Actions with GitHub-hosted runners to build and deploy our application.

This application is delivered as a Docker image. A custom base image is used to build the deliverable image.

## The GitHub side

As previously mentioned, we'd need to store the base image somewhere close to your GitHub runners, because we need to pull it as a part of our build process. It makes sense to keep the base image in GitHub Container Registry, simply because this traffic will be free. Also, if we want to improve the build time, we can use the GitHub cache action to cache the base image.

However, the final image should be pushed to Azure Container Registry. We won't be charged for igress traffic.

This way we can avoid charges for the traffic betrween GitHub Container Registry and Action runners.

## Cloud side

Our Kubernetes manifests should reference the images stored in the ACR. This way can keep the thraffic free, by co-locating the ACR and the Kubernetes cluster in the same region. And at the same time we wil avoid pulls between the cluster and GitHub Container Registry.

## Conclusion

Both external and internal container registries have their purpose and it's perfectly fine to use both at the same time.

## Disclosure

> I'm employed by GitHub at the time of writing this post. All opinions are my own.