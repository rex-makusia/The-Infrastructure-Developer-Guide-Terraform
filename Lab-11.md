# Keeping Terraform Remote State DRY with Terragrunt in Azure

## Description
As Terraform configurations grow to be large and complex, the code becomes harder to manage and make changes. Terragrunt is an open-source tool that acts as a thin wrapper around Terraform, enabling configurations to be more DRY. DRY is a popular software development practice that stands for "Don't Repeat Yourself," which means don't re-use the same software pattern, avoiding redundancy. Terragrunt adds a feature that allows Terraform remote state configurations to be defined once in code and re-used throughout multiple Terraform projects.

In this lab, you will use Terragrunt to deploy a Terraform configuration in separate environments while keeping the remote state configuration DRY.

Terragrunt is a thin wrapper around Terraform that provides additional tools for keeping configurations DRY. DRY is a popular software development practice that stands for "Don't Repeat Yourself," which means don't reuse the same software pattern, avoiding redundancy. Terragrunt enables developers to use DRY principles when configuring remote state. For information on how to install Terragrunt, check out the 
[install documentation](https://terragrunt.gruntwork.io/docs/getting-started/install/#install-terragrunt)

In this lab step, you will use Terragrunt to deploy a Terraform configuration in separate environments while keeping the remote state configuration DRY.

### Learning Objectives
Upon completion of this lab you will be able to:

Understand how to deploy Terraform resources with Terragrunt
Learn how to keep the remote state configuration DRY using Terragrunt

### Intended Audience
This lab is intended for:

- Individuals familiar with using Terraform and how to deploy resources
- Anyone interested in learning how to use Terragrunt to increase Terraform effectiveness
- Enterprises looking to scale their Terraform codebase

### Lab Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform modules and variables
  
The following course and lab can be used to fulfill the prerequisites:

- Introduction to DevOps
- Creating Reusable Infrastructure with Terraform Modules in Azure
- Using Terraform Remote State in Azure

#### Azure login
```az
az login
```

#### Project hierarcy
```yaml
dev
  -> main.tf
  -> terragrunt.hcl
prod
  -> main.tf
  -> terragrunt.hcl
qa
  -> main.tf
  -> terragrunt.hc
terragrunt.hcl
```

dev
```maint.tf```
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}

#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet-dev-westus-001"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "REPLACEME"
}

# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet-dev-westus-001"
  resource_group_name  = "REPLACEME"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  address_prefixes     = ["10.0.0.0/24"]
}
```

```terragrunt.hcl```
```hcl
# Include all settings from the root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}
```
prod
```main.tf```
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}

#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet-prod-westus-001"
  address_space       = ["10.1.0.0/16"]
  location            = "westus"
  resource_group_name = "REPLACEME"
}

# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet-prod-westus-001"
  resource_group_name  = "REPLACEME"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  address_prefixes     = ["10.1.0.0/24"]
}
```

```terragrunt.hcl```
```hcl
# Include all settings from the root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}
```
qa
```maint.tf```
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}

#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet-qa-westus-001"
  address_space       = ["10.2.0.0/16"]
  location            = "westus"
  resource_group_name = "REPLACEME"
}

# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet-qa-westus-001"
  resource_group_name  = "REPLACEME"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  address_prefixes     = ["10.2.0.0/24"]
}
```
```terragrunt.hcl```
```hcl
# Include all settings from the root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}
```
```terragrunt.hcl```
```hcl
# Remote backend settings for all child directories
remote_state {
  backend = "azurerm"
  config = {
    resource_group_name  = "REPLACEME"
    storage_account_name = "STORAGEACCOUNT"
    container_name       = "calab" 
    key                  = "${path_relative_to_include()}/terraform.tfstate"
  }
}
```
#### Find terraform files
```sh
find "./terraformlab" -name "*.tf" | xargs sed -i "s/REPLACEME/cal-976-8e4/g"
find "./terraformlab" -name "*.hcl" | xargs sed -i "s/REPLACEME/cal-976-8e4/g"
find "./terraformlab" -name "*.hcl" | xargs sed -i "s/STORAGEACCOUNT/sacalabscal9768e4/g"
```

```terragrunt.hcl```
```hcl
# Remote backend settings for all child directories
remote_state {
  backend = "azurerm"
  config = {
    resource_group_name  = "cal-976-8e4"
    storage_account_name = "sacalabscal9768e4"
    container_name       = "calab" 
    key                  = "${path_relative_to_include()}/terraform.tfstate"
  }
}
```

#### Terragrunt Run
```hcl
terragrunt run-all apply

```
