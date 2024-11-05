
# The Full Stack Team Infrastructure

## Introduction
This repository contains the main infrastructure project for **The Full Stack Team** application, built with Terraform and Terragrunt. The Terraform configurations are organized into reusable modules, while Terragrunt keeps the environment-specific configurations DRY.

## Security and Best Practices
Since this is a public repository, follow these guidelines to ensure security:
1. **Sensitive Data**: Store all sensitive data, such as passwords, tokens, and secrets, in [Azure Key Vault](https://docs.microsoft.com/en-us/azure/key-vault/) and not directly in the files.
2. **Terraform State**: Terraform state files are stored in an Azure Storage Account to prevent accidental uploads. Make sure that `terraform.tfstate` is ignored in `.gitignore`.
3. **Environment Variables**: Set up environment variables for local testing instead of storing secrets directly in configuration files.

## Terraform & Terragrunt
All Terraform modules are located in the `_modules_` directory. We use [Terragrunt](https://terragrunt.gruntwork.io/) to manage configurations per environment and to handle state backends and variables consistently.

Run Terragrunt commands instead of Terraform. For example:
- Instead of `terraform plan`, use `terragrunt plan`
- Instead of `terraform apply`, use `terragrunt apply`

## Setting Up Azure Backend for State Management
For each environment, a separate Azure Storage Account is set up to store Terraform state remotely. Ensure that backend configurations are consistent across environments.

### Azure Backend Initialization Script
Use the `initialize_azure_backend.sh` script to set up backend storage:

```bash
bash scripts/initialize_azure_backend.sh <your-azure-subscription-id>
```

This script creates:

- An Azure resource group for backend storage
- A storage account and blob container for Terraform state files

The script automatically applies settings that match the backend configurations specified in terragrunt.hcl.

### Backend Configuration Example
In `envs/terragrunt.hcl`, ensure backend configuration matches the storage resources created above:

```hcl
remote_state {
  backend = "azurerm"
  config = {
    resource_group_name  = "tfst-tfstate-dev"
    storage_account_name = "tfstdevstate"
    container_name       = "tfstate-container"
    key                  = "${path_relative_to_include()}/terraform.tfstate"
  }
}
```

## Environment Structure
Each environment has its own configuration folder in `envs/` (e.g., `dev`, `staging`, `prod`). Each environment uses Terragrunt to handle environment-specific variables and module dependencies.

### Example for `dev` Environment
- Folder structure:
  ```plaintext
  infrastructure/
  ├── envs/
  │   └── dev/
  │       ├── core/
  │       │   └── terragrunt.hcl
  │       └── storage/
  │           └── terragrunt.hcl
  ```
- `terragrunt.hcl` files reference modules and set up environment-specific variables.

## Running Terragrunt Locally
To apply changes locally, navigate to the environment folder (e.g., `envs/dev/storage`) and use `terragrunt plan` or `terragrunt apply` as follows:

```bash
cd envs/dev/storage
terragrunt plan -out tfst.tfplan
terragrunt apply tfst.tfplan
```

## Module Dependencies
Use Terragrunt to visualize dependencies within an environment. Install [GraphViz](https://graphviz.org/) and run:

```bash
cd envs/dev
terragrunt graph-dependencies | dot -Tpng > dependencies.png
```

## FAQs
### Error: State Lock
If you encounter a lock error, check if a pipeline was interrupted or another user has the lock. Use the following command to inspect the lock:

```bash
az storage blob show --container-name tfstate-container --name dev/core/terraform.tfstate --account-name tfstdevstate | jq '.properties.lease'
```

To force unlock, use:

```bash
terragrunt force-unlock <lock-id>
```

### Error: Resource Already Exists
This error can occur if a resource was created manually or outside of Terraform. Use `terragrunt import` to import the existing resource:

```bash
terragrunt import <resource> <resource-id>
```

## Contributing
Follow the guidelines in `CONTRIBUTING.md` for information on contributing to this project.