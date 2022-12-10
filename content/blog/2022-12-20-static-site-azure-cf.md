---
date: '2022-12-20T00:00:00Z'
title: 'End-to-End Static Site on Azure with Cloudflare'
tags:
  - azure
  - cloudflare
  - devops
  - terraform
slug: static-site-azure-cf
---

In this post, I will show you how to set up a static Hugo site on Azure with a custom domain managed by Cloudflare. The site will be built by Hugo, served from Azure Storage. It will be deployed by GitHub Actions.

## Prerequisites

To follow this tutorial, you will need the following:
* Azure subscription
* Cloudflare account
* GitHub account

## Repository

You can find the repository for this tutorial [here](https://github.com/asizikov/static-website-ci).

`infra` directory contains Terraform code to create Azure resources. `site` directory contains the Hugo site.

## Create Azure resources

First, we need to create a resource group and a storage account. We will use Terraform to create the resources. Create a new file called `main.tf` in the `infra` directory of the repository. Add the following content to the file:


## Create a static site


## Create a GitHub repository

Next, we need to create a GitHub repository. You can use any repository that you want. In this tutorial, I will use a repository called `static-site`. You can create a new repository by clicking on the `New` button in the GitHub UI:

![New repository](/images/2022-12-static-site-azure-cf/new-repository.png)

Next, enter the name of the repository and click on `Create repository`:

![Create repository](/images/2022-12-static-site-azure-cf/create-repository.png)

## Create a GitHub Actions workflow

Next, we need to create a GitHub Actions workflow. We will use the Azure CLI to deploy the site to Azure. Create a new file called `deploy.yml` in the `.github/workflows` directory of the repository. Add the following content to the file:

```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:

    deploy:
        runs-on: ubuntu-latest
        steps:
        - uses: actions/checkout@v2
        - name: Deploy to Azure
            uses: azure/cli@v1
            with:
            azcliversion: 2.29.2
            inlineScript: |
                az login --service-principal -u ${{ secrets.AZURE_CLIENT_ID }} -p ${{ secrets.AZURE_CLIENT_SECRET }} --tenant ${{ secrets.AZURE_TENANT_ID }}
                az storage blob service-properties update --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} --static-website --404-document 404.html --index-document index.html
                az storage blob upload-batch --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} --destination \$web --source _site
    ```

## Create a service principal

Next, we need to create a service principal. We will use the service principal to authenticate to Azure. Open a terminal and run the following commands:

```bash

az ad sp create-for-rbac --name static-site --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/static-site --sdk-auth

```

Replace `<SUBSCRIPTION_ID>` with the ID of your Azure subscription. The command will output the following:

```json
{
  "clientId": "<CLIENT_ID>",
  "client: