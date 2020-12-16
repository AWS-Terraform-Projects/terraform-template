# An Template Repository using GitHub Actions and Terraform

> A template repository for provisioning AWS resources using Terraform. The template supports both Terraform Cloud or AWS S3 to manage the the project's remote files. GitHub Actions is used to implement the CI/CD pipeline


### Installing and running this project
<details>
  <summary>Using this template repository</summary>

  #### Choose the `Use this template` to create a new repository using this template repository as shown below
  
  <kbd><img src="./documentation/use-terraform-template.png" /></kbd>
</details>

<details>
  <summary>Prerequisites for running this project</summary>
  
#### The project has the following dependencies  
- AWS CLI version 2. To install the AWS CLI, please see [Installing, updating, and uninstalling the AWS CLI version 2](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- Terraform CLI 0.14.2 . To install Terraform CLI, please see [Install Terraform](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started)
</details>

<details>
  <summary>Initialize and run this project</summary>
  
#### Initialize Terraform using S3 as the remote backend
> To intialize Terraform using S3 as the remote backend, edit the `backend_s3.hcl` file, replacing the below properties:
>
> bucket = "S3 bucket name"
>
> key    = "Path to the terraform.tfstate"
>
> region = "AWS region where this bucket is located"
>
> After the `backend_s3.hcl` file has been updated with the S3 bucket details, edit the `main.tf' file to use 's3' as the remote backend
>
>
> Now run the below command to initialize S3 with the terraform state files:
> ```bash
> terraform init -backend-config=backend_s3.hcl
> ```

#### Initialize Terraform using Terraform Cloud as the remote backend
> To intialize Terraform using Terraform Cloud as the remote backend, edit the `backend.hcl` file, replacing the below properties:
>
> workspaces { name = "Name of your Terraform Cloud Workspace" }
>
> organization = "Name of your Terraform Cloud organization"
>
> *For instructions on setting up your free Terraform Cloud Account see [Getting Started with Terraform Cloud](https://learn.hashicorp.com/collections/terraform/cloud-get-started)* 
> 
> | Set the below 3 properties as environment variables in your Terraform Cloud workspace|
> | -------------------------------------------------------------------------------------|
> | AWS_DEFAULT_REGION |
> | AWS_SECRET_ACCESS_KEY (**Use the sensitive checkbox to protect this value!!**) |
> | AWS_ACCESS_KEY_ID (**Use the sensitive checkbox to protect this value!!**) |
>
> 
> After the `backend.hcl` file has been updated with the Terraform Cloud's organization and workspace details, run the below command to login to your Terraform cloud workspace
>
> ```bash
> terraform login
> ```
>
> After a successful login, initialize your Terraform Cloud workspace with the terraform state files:
> ```bash
> terraform init -backend-config=backend.hcl
> ```

#### Run the standard Terraform workflow commands
>
> To validate your terraform project, run the below command
>
> ```bash
> terraform validate
> ```
>
> To review the final plan and terraform changes before applying them, run the below command
>
> ```bash
> terraform plan
> ```
>
> To apply the changes and provision the AWS API Gateway, run the below command
>
> ```bash
> terraform apply
> ```
>
> To clean-up and delete the provisioned AWS resources, run the below command
>
> ```bash
> terraform destroy
> ```
</details>

<details>
  <summary>Configure GitHub Actions</summary>

  #### Configure GitHub Actions to use S3 as the remote backend
>
> To configure GitHub Actions to use S3, the below 2 properties needs to be added as *GitHub Secrets*
>
> | GitHub Secrets for AWS S3 remote backend
> | ----------------------------------------
> | AWS_ACCESS_KEY_ID
> | AWS_SECRET_ACCESS_KEY
>
>> To configure GitHubs secretes, please see [GitHub Encrypted secrets](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets)
>
  #### Configure GitHub Actions to use Terraform Cloud as the remote backend
>
> To configure GitHub Actions to use Terraform Cloud, the below property needs to be added as *GitHub Secrets*
>
> | GitHub Secrets for Terraform Cloud remote backend
> | ----------------------------------------
> | TF_API_TOKEN
>
>> To generate a Terraform Cloud Team API Token , please see [Terraform Cloud Team API Token](https://www.terraform.io/docs/cloud/users-teams-organizations/api-tokens.html)
>
</details>
