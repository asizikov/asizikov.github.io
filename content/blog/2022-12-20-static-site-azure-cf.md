---
date: '2022-12-20T00:00:00Z'
title: 'End-to-End Static Site on Azure with Cloudflare'
tags:
  - azure
  - cloudflare
  - devops
slug: static-site-azure-cf
---

In this post, I will show you how to set up a static site on Azure with Cloudflare. The site will be served from Azure Storage and will be protected by Cloudflare. The site will be deployed from GitHub Actions. The whole process will take about 10 minutes.

## Prerequisites

To follow this tutorial, you will need the following:
* Azure subscription
* Cloudflare account
* GitHub account

## Create Azure resources

First, we need to create a resource group and a storage account. We will use the Azure CLI to do that. Open a terminal and run the following commands:

```bash
az group create --name static-site --location westeurope
az storage account create --name staticsite --resource-group static-site --sku Standard_LRS --encryption-services blob
```

## Create a static site

Next, we need to create a static site in the storage account. We will use the Azure Portal for that. Open the Azure Portal and navigate to the storage account that you created in the previous step. Click on `Static website` in the left menu and then click on `Enable`:

![Enable static website](/images/2022-12-static-site-azure-cf/enable-static-website.png)

Next, click on `Save`:

![Save static website](/images/2022-12-static-site-azure-cf/save-static-website.png)

Now, you should see the following:

![Static website enabled](/images/2022-12-static-site-azure-cf/static-website-enabled.png)

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