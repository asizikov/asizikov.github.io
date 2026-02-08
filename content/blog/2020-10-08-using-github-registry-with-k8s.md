---
date: "2020-10-08T00:00:00Z"
image: /images/2020/10-ghk8s/helmsman2.jpg
title: Using GitHub Container Registry with Kubernetes
tags: 
    - devops
    - kubernetes
    - github
slug: using-github-registry-with-k8s     
---

GitHub Container Registry was introduced on the 1st of September 2020. It's still in the Beta stage, so it's rather not recommended to use it in production. However, it offers us free private storage for our Docker images, at least until the end of the Beta period. 

Private storage, free and unlimited download... looks like a good enough option for local development. 

In this post, I'm going to configure my local Kubernetes cluster to pull images from my GitHub Container Registry.

![Arthur John Trevor Briscoe, Helmsman](/images/2020/10-ghk8s/helmsman2.jpg)

## Enable GitHub Container Registry

First things first. We need to enable GitHub Container Registry for our account (Process for your organization is almost the same).

> Note GitHub Container Registry is in public beta state at the moment of writing, so it's subject to change. I recommend to follow [the official documentation](https://docs.github.com/en/free-pro-team@latest/packages/getting-started-with-github-container-registry), it's more likely to be up to date.

![](/images/2020/10-ghk8s/registry.jpg)

## Create Access Tokens


For now, Container Registry supports just one authorization option, which is Personal Access Token (PAT). I'm going to [create two tokens here](https://github.com/settings/tokens) (note the scopes).

![](/images/2020/10-ghk8s/pat-tokens.png)

One token with `read:packages` scope will be used for my local Kubernetes cluster to perform pull actions. And another token I'll use to manage images in my registry (remember Authentication step from GitHub Packages configuration page?).

## .dockerconfigjson

Once I have my PAT token created I can build an auth string using the following format: `read:packages` scope. 


`username:123123adsfasdf123123` where `username` is my GitHub username and `123123adsfasdf123123` is the token with read scope.

let's Base64 encode it first:

```bash
echo -n "username:123123adsfasdf123123" | base64
dXNlcm5hbWU6MTIzMTIzYWRzZmFzZGYxMjMxMjM=
```

With this string I can compose a new `.dockerconfigjson`:

```json
{
    "auths":
    {
        "ghcr.io":
            {
                "auth":"dXNlcm5hbWU6MTIzMTIzYWRzZmFzZGYxMjMxMjM="
            }
    }
}
```

Which we can  Base64 encode again:

```bash
echo -n  '{"auths":{"ghcr.io":{"auth":"dXNlcm5hbWU6MTIzMTIzYWRzZmFzZGYxMjMxMjM="}}}' | base64
eyJhdXRocyI6eyJnaGNyLmlvIjp7ImF1dGgiOiJkWE5sY201aGJXVTZNVEl6TVRJellXUnpabUZ6WkdZeE1qTXhNak09In19fQ==
```

and store it at as `dockerconfigjson.yaml` file.

{{< gist asizikov d620e31e9cac51187ccc73ddd8189e3d >}}

> Note: base64 is not an encryption, so you should not commit this file! This is just for a demo.

And now we have prepared all the pieces, time to create a new secret

```bash
kubectl create -f dockerconfigjson.yaml
secret/dockerconfigjson-github-com created
```

Which can be used later on when we schedule our pods:

{{< gist asizikov 3fc9e123ccce5c05322fb85464b29b01 >}}

And the final step is to check Pod Events (I'm using [Lens Kubernetes IDE here](https://k8slens.dev/)).

![](/images/2020/10-ghk8s/image-pulled.png)


Have fun, and enjoy it while it's free!