# Overview
This wiki page provides instructions on how to initialize a new Terraform repository using this template repository. By using this template repository as a base, the new repository will inherit the following functionalities :

1. A GitHub Actions CI/CD workflow that deploys to AWS
2. A Terraform project structure that follows the best practices as recommended by Hashicorp

## Create a new repository
Start by creating a new GitHub repository by using this template repository. Click on the **Use this template** button as shown below
  
![](https://github.com/AWS-Terraform-Projects/terraform-template/blob/master/documentation/use-terraform-template.png)

## Configure a remote backend for Terraform
`Terraform` is a stateful build tool that needs to create and access its state across multiple deployments. These state files are used by `Terraform` to manage the resources it has created. While testing on local workstations, this files are created in the developer's workstation. But while running in a CI/CD pipeline on remote servers, these state files have to be persisted in a safe and consistent data store. This template repository supports the following two remote data stores:
1. An AWS S3 bucket
2. A (free) Terraform Cloud workspace

## Configure S3 as an remote back-end for Terraform

#### Create a S3 bucket in AWS
Create a new S3 bucket in AWS for storing the state files 'Terraform' creates. The S3 bucket should be private so that only the AWS account with the correct credentials can access the files stored in this bucket.
Once the bucket has been created, create a folder in the bucket. This folder will be used as the **key** by `Terraform` for storing its files. 

Each `Terraform` project should store its files in a separate folder or **key** in this S3 bucket. Turn on versioning for the bucket so that the files in this S3 bucket are version-controlled

At the end of this step, we should have two values needed for the next step:
1. The S3 bucket name
2. The folder or **key** for storing `Terraform` files within the S3 bucket
---
 
#### Edit backend_s3.hcl
The `backend_s3.hcl` file is used by `Terraform` to store its remote state in a S3 bucket. The file has 3 properties that need to be set:
1. bucket - the name of the S3 bucket used for storing `Terraform` state files
2. key - The folder in the S3 bucket where this project's files needs to be stored
3. region - The AWS region where the S3 bucket was created.

Edit the `backend_s3.hcl` file with details specific to your S3 bucket. A sample `backend_s3.hcl` is shown below:
```hcl
bucket = "terraform-deployments"
key    = "api-gateway/terraform.tfstate"
region = "us-east-2"
```

After the backend_s3.hcl file has been updated with the S3 bucket details, edit the main.tf file to use 's3' as the remote backend. Please see sample code below:

```hcl
terraform {
  backend "s3" {}

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 2.70"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Sample module that provisions an AWS API Gateway. Replace with the module that configures 
# the specific AWS resource you wish to provision 
module "http_api_gateway" {
  source                       = "./modules/http-api-gateway"
  http_api_gateway_name        = var.http_api_gateway_name
  http_api_gateway_description = var.http_api_gateway_description
}

```

---

#### Edit terraform.yml to use the backend_s3.hcl as the remote backend
The `terraform.yml` file is the GitHub Actions workflow that contains instruction for the CI/CD pipeline. This file is used by the CI/CD workflow to execute `Terraform` workflow and provision AWS resources.

The `terraform.yml` file is located under the `.github/workflows` folder. Edit the job that initializes `Terraform` as shown below:
```hcl
    # Initialize a new or existing Terraform working directory by creating initial files, 
    # loading any remote state, downloading modules, etc.
    - name: Terraform Init
      run: terraform init -backend-config=backend_s3.hcl
```
The `terraform init -backend-config=backend_s3.hcl` command instructs `Terraform` to use the `backend_s3.hcl` to initialize the S3 bucket with its remote state

---

#### Create GitHub Secrets for AWS credentials
Since we are using a private S3 bucket to store `Terraform's` remote state, we need to provide the AWS credentials to `Terraform's` CLI. We do this in a secure manner by creating GitHub Secrets.

Create the below 2 GitHub Secrets:
1. AWS_ACCESS_KEY_ID
2. AWS_SECRET_ACCESS_KEY 

> _For Instructions on how to create GitHub Secrets see [Encrypted secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets)_

This completes the `Terraform-S3` configuration. The GitHub Actions CI/CD Pipeline should now be all configured to use S3.

Create a Pull Request to trigger the CI/CD workflow. You should be able to see the GitHub Workflow execute under the **Actions** tab as shown below:

  <kbd>![](https://github.com/AWS-Terraform-Projects/terraform-template/blob/master/documentation/GitHub-Workflow-S3.png)</kbd>


---

## Configure Terraform Cloud as an remote back-end for Terraform

As an alternative to S3, Terraform offers a free Cloud account for storing its remote state. While S3 acts simply as a remote data store, Terraform offers much more functionality than a simple data store. Some of features Terraform cloud offers are:
1. Locking of the files so that just one process can update the files
2. Team access. Each workspace can be assigned to a team
3. Enhanced security
4. Cost estimate. Terraform Cloud offers an estimate of the cost for provisioning the cloud resources found in the configuration. This is a paid feature

The above are just a few of the features that Terraform Cloud offers over a generic S3 data store

#### Create a free Terraform Cloud account

Start by creating your free Terraform Cloud account. Details on how to create and configure your Terraform Cloud account is available at [Sign up for Terraform Cloud](https://learn.hashicorp.com/tutorials/terraform/cloud-sign-up?in=terraform/cloud-get-started)

---

#### Create a Terraform organization and workspace 

After signing up for your Terraform Cloud account, create an organization and a workspace for that organization. For details on setting this up, please see [Create a workspace](https://learn.hashicorp.com/tutorials/terraform/cloud-workspace-create?in=terraform/cloud-get-started)

---

#### Add AWS credentials as Environment variables in the Terraform organization

Since we'll be provisioning AWS resources using Terraform, we have to configure Terraform cloud to use AWS keys. To see how to do this, please reference [Configure a Workspace and Create Infrastructure](https://learn.hashicorp.com/tutorials/terraform/cloud-workspace-configure?in=terraform/cloud-get-started)

---

#### Generate a Terraform Team API token

We now need to generate a Terraform Team token. This token will allow GitHub Actions to securely access your Terraform Cloud account and execute the Terraform workflow as well as store remote state files. 

To generate a Terraform Team token , please see [Team API Tokens](https://www.terraform.io/docs/cloud/users-teams-organizations/api-tokens.html#team-api-tokens)

We are now done with configuring Terraform Cloud. We will now configure GitHub Actions workflow to use the remote Terraform cloud account 

---

#### Add the Terraform Team API token as a GitHub Secret

Using the Terraform Team token created above, add a new GitHub Secret named `TF_API_TOKEN`

> _For Instructions on how to create GitHub Secrets see [Encrypted secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets)_

---

#### Edit backend.hcl with the Terraform organization and workspace details

Edit the backend.hcl file with your Terraform Cloud account's organization and workspace name. A sample backend.hcl file is shown below:
```hcl
workspaces { name = "my-team-workspace" }
organization = "my-terraform-cloud-org"
```

---

#### Edit terraform.yml to use the backend.hcl as the remote backend

The `terraform.yml` file is the GitHub Actions workflow that contains instruction for the CI/CD pipeline. This file is used by the CI/CD workflow to execute `Terraform` workflow and provision AWS resources.

The `terraform.yml` file is located under the `.github/workflows` folder. Edit the job that initializes `Terraform` as shown below:
```hcl
    # Initialize a new or existing Terraform working directory by creating initial files, 
    # loading any remote state, downloading modules, etc.
    - name: Terraform Init
      run: terraform init -backend-config=backend.hcl
```
The `terraform init -backend-config=backend.hcl` command instructs `Terraform` to use the `backend.hcl` to initialize the Terraform Cloud with its remote state


This completes the `Terraform Cloud` configuration. The GitHub Actions CI/CD Pipeline should now be all configured to use Terraform Cloud.

---

## Terraform workflow

#### If using Terraform Cloud, run the below command to login

 ```bash
 terraform login
 ```

#### Run the standard Terraform workflow commands

 To validate your terraform project, run the below command

 ```bash
 terraform validate
 ```

 To review the final plan and terraform changes before applying them, run the below command

 ```bash
 terraform plan
 ```

 To apply the changes and provision the AWS resources, run the below command

 ```bash
 terraform apply
 ```

 To clean-up and delete the provisioned AWS resources, run the below command

 ```bash
 terraform destroy
 ```
 ---

