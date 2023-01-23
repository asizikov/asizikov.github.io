---
date: "2023-01-23T00:00:00Z"
title: GitHub Container Registry. Keeping traffic below the spending limit.
tags: 
 - github-actions
 - azure
 - devops
slug: ghcr-acr-traffic
---

GitHub Container Registry is a great service. It's free for public repositories and it's integrated with GitHub Actions. Unfortunately, it's not free for private packages. The free tier allows 1 GB of data transfer per month.
What happens when you reach the limit depends on your payment profile. You will either be prevented from downloading your images or you'll be charged for the consumed traffic.

The up-to-date pricing and details can be found [here](https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages#about-billing-for-github-packages).

Paid and enterprise customers will have different limits and prices. Anyway, it's still a good idea to be cautious of the data transfer limits and be able to save some money.

The other day I had a conversation with a GitHub user on Mastodon (I know, I know) and he was confused by the traffic limits and the logic behind them. I decided to write a post about it.

## How does the traffic limit work?

All data transferred in from any source is free. And the data transferred out is metered. However, if you pull an image using a `GITHUB_TOKEN`, the traffic is not counted. So in other words pulls that you make as a part of your CI/CD pipeline (via GitHub Actions) are not counted.

A typical use case would be the pull of custom base images or an image that you use to execute certain tasks as a part of your pipeline.

On the other hand, if you pull an image from outside of GitHub (e.g. from your Kubernetes cluster), the traffic is counted.

![Free Pull vs Metered](/images/2023-01-ghcr-acr/free-pull-vs-metered.png)

In the diagram above green pulls are free while the red ones are metered.

At the same time if you use an external registry (e.g. Azure Container Registry) your cluster will be able to pull images without any traffic limits. Of course, if your cluster is located in the same region as your ACR. But at the same time, if you pull images as a part of your CI/CD pipeline, you'll be charged for egress traffic. See the diagram below.

![Free Pull vs Metered 02](/images/2023-01-ghcr-acr/free-pull-vs-metered-02.png)

So, that seems that either way you'll be charged. But there is a way to avoid it.

## The GitHub side

As previously mentioned, we'd need to store the base image somewhere close to our GitHub runners, because we need to pull it as a part of our build process. It makes sense to keep the base image in GitHub Container Registry, simply because this traffic will be free. Also, if we want to improve the build time, we can use the GitHub cache action to cache the base image.

However, the final image should be pushed to Azure Container Registry. We won't be charged for ingress traffic.

This way we can avoid charges for the traffic between GitHub Container Registry and Action runners.

## Cloud side

Our Kubernetes manifests should reference the images stored in the ACR. This way we avoid traffic charges, by co-locating the ACR and the Kubernetes cluster in the same region. And at the same time, we will avoid pulls between the cluster and GitHub Container Registry.

The final architecture will look like this:

![Architecture](/images/2023-01-ghcr-acr/architecture-02.png)

## Conclusion

Both external and internal container registries have their purpose and it's perfectly fine to use both at the same time.

In this scenario, the GitHub Container registry serves the CI/CD process and stores images needed for the build process.

Azure Container Registry stores the final images and is used by the Kubernetes cluster.

## Notes

In this post I'm not discussing storage prices, this is a separate story.

## Disclosure

> I'm employed by GitHub at the time of writing this post. All opinions are my own.