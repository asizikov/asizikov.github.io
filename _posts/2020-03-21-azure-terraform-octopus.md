---
layout: post
title: Provision your Azure environment with Terraform and Octopus Deploy
image: /images/2020-03-terraform-octopus/result.JPG
---

My current project has reached the point where we have to manage our infrastructure in a more organized way rather than ad-hoc manual configurations.

### Current landscape

We use a pretty standard set up for a .NET web application.

We keep our code in one mono repo hosted in internal GitLab. We use TeamCity to build our code and we use Octopus Deploy to run our deployments to Azure.

We have several environments (let's call them `feature`, `test` and `prod`) and they all are different.

![Current setup](/images/2020-03-terraform-octopus/current-setup.JPG)

### Scope of the problem

We'd like to move to an Infrastructure-as-a-Code approach and provision our system automatically as a part of our CI/CD pipeline.

To do that we're going to add [Terraform](https://www.terraform.io/) to the game. At the time I'm writing this post the versions for the software are following:

* TeamCity 2019.2.2
* Octopus Deploy v2019.12.0 LTS
* Terraform v0.12.24
* `azurerm` provider 2.2.0

### Repository Setup

This part is not very important, but in order to provide you with the full picture I'll put it here.

We keep your Terraform files in the same repository alongside the application code and database schemas. Basically it's just an `\infrastructure` directory in the root of the repo.

### Terraform Setup

We run our application in Azure, hence the choice of Terraform backend and provider: we keep [Terraform state](https://www.terraform.io/docs/state/index.html) in Azure. 

Please refer to [the official documentation](https://docs.microsoft.com/en-us/azure/terraform/terraform-backend) for a detailed step by step guide on setting up a storage for Terraform State.

In short you'd need to run the following:

```bash
#!/bin/bash

RESOURCE_GROUP_NAME=terraformstate-rg
STORAGE_ACCOUNT_NAME=terraformstateci
CONTAINER_NAME=terraformstate

# Create resource group
az group create --name $RESOURCE_GROUP_NAME --location eastus

# Create storage account
az storage account create --resource-group $RESOURCE_GROUP_NAME --name $STORAGE_ACCOUNT_NAME --sku Standard_LRS --encryption-services blob

# Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group $RESOURCE_GROUP_NAME --account-name $STORAGE_ACCOUNT_NAME --query [0].value -o tsv)

# Create blob container
az storage container create --name $CONTAINER_NAME --account-name $STORAGE_ACCOUNT_NAME --account-key $ACCOUNT_KEY

echo "storage_account_name: $STORAGE_ACCOUNT_NAME"
echo "container_name: $CONTAINER_NAME"
echo "access_key: $ACCOUNT_KEY"
```

Once the you've successfully created a storage for your state file, you can update your `main.tf` file with the backend configuration: 

{% gist 06893cb889d885d7e749a5444fb00b14 %}

For the purpose of this blog post I'll be creating just a new resource group with one SQL server. 

Let's declare our set up in a new `sql.tf` file: 

{% gist ff7e182aa9ff919b394c22d4dcae6795 %}

note that I'm using variables in a gist above. 

Variables are defined in a `variables.tf` file: 

```HCL
variable "sql_server_admin_pwd" {
  type    = string
}
```

This works fine, but for the local testing I keep the `terraform.tfvars` files with the values: 

```HCL
resource_group_name  = "test_rg"
sql_server_admin_name = "admin_name"
sql_server_admin_pwd = "P@$$w0rd1!" # dummy pwd used for local tests only

```

values are not encrypted, but they are just for the local tests and experiments. They will be replaced with the environment specific vars by Octopus later.

### TeamCity Setup

Octopus has a specific syntax for variable substitution. Usually it looks for `#{Variable}` markers in your config files. As you can imagine they are not always valid as a resource name.

As I mentioned before our `terraform.tfvars` has realistic values for local tests. It helps our infrastructure engineers for verify their work by running `terraform plan` commands.

However when we build our packages we want to convert `tfvars` file to replace variable's values with the syntax suitable for Octopus. 

I had to invent a convention here.  Every variable declaration `var_name = "abc123"` will be converted to a  `var_name = "#{var_name}"` line.

To do that we've set up a build step with the following PowerShell script:

{% gist ff5aa7c6bd6a7f513e84622dc9fcfbea %}

After that we can build our code and then create a release package with our Terraform config.

A TeamCity build configuration has a custom `%octopus.package.suffix%` variable declared. This is a suffix which is composed from the branch name. It's empty for the master branch build, and has something like `feature-AB-123` for a feature branch with a `AB-123` name. Yes, this is a JIRA ticket name :) 

With this variable we can prepare a zipped artifacts file: 

```
+:infrastructure/** => Terraform.%build.number%%octopus.package.suffix%.zip
```

that would produce a file `Terraform.1.123.9-feature-AB-123` which will later be uploaded to Octopus.

![TeamCity pipeline](/images/2020-03-terraform-octopus/team-city-pipeline.JPG)

### Octopus Setup

Let's create a new Step in Octopus project: 

![Octopus Step](/images/2020-03-terraform-octopus/octopus-step.png)

We currently use an official [Apply a Terraform template](https://octopus.com/docs/deployment-examples/terraform-deployments/apply-terraform). 

However there are few things which we had to configure in order to make it work for us. 

First of all Octopus cannot keep it up with Terraform development, so you may need to update a `terraform.exe` version yourself. Just grab a latest version from [Terraform Downloads page](https://www.terraform.io/downloads.html) and put in a folder on your Octopus server. Something like `C:\Program Files\Terraform` would do.

Then you need to provide a path to you custom `terraform.exe` executable. To do that you need to set up a variable: `Octopus.Action.Terraform.CustomTerraformExecutable`. Give it a `C:\Program Files\Terraform\terraform.exe` value.

As you remember the release package contains a `terraform.tfvars` file with templates in it. So don't forget to tick this box:

![Octopus Step](/images/2020-03-terraform-octopus/octopus-replace.png)

And of course you will have to define your variables for each environment:

![Octopus Step](/images/2020-03-terraform-octopus/octopus-variables.png)

And now we have a new player in our CI/CD pipeline: 

![Octopus Step](/images/2020-03-terraform-octopus/result.JPG)