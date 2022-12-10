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

`infra` directory contains Terraform code, and `site` directory contains the Hugo site.

## Create Azure resources

I'm going to use azure storage as a backend for terraform. You can use any backend you want. To create a storage account, run the following command:

```bash
# Create a resource group that will contain the resources

az group create --name <RESOURCE_GROUP_NAME> --location <LOCATION>
# Create a storage account that will be used as a backend for Terraform

az storage account create --name <STORAGE_ACCOUNT_NAME> --resource-group <RESOURCE_GROUP_NAME> --sku Standard_LRS --encryption-services blob
```

We're going to use OIDC authentication to authenticate [GitHub workflows to Azure](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-azure). To do that, we need to create an Azure AD application. We'll follow the instructions from the [Azure documentation](https://learn.microsoft.com/en-us/azure/developer/github/connect-from-azure?tabs=azure-portal%2Clinux).

Make sure to follow the least privileged principle and assign the application only the necessary permissions. In our case I'd recomment limiting the scope of the application to the resource group that will contain the resources.

Once we have it all prepared, we can maintain the rest with Terraform and Terragrunt.

Project structure:



## Create a static site


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