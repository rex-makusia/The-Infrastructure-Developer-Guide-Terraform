# Creating Reusable Infrastructure with Terraform Modules in Azure

## Description
Modules are a critical component of production-grade Terraform configurations. It gives infrastructure developers the ability to split infrastructure services up into separate components. For example, you can have a module for deploying Virtual Machines and a module for Virtual Networks. You can then use each module as a building block for creating entire environments in Azure.

Terraform configuration files that are thousands of lines of code are considered an anti-pattern. Infrastructure code becomes unmanageable, and execution plans can take over 10 minutes to create. Instead, split the code up into modules. Modules separate pieces of the infrastructure into reusable, testable units.

Modules also provide a way for the community to share Terraform configurations with others. There are hundreds of modules on the [Terraform Registry](https://registry.terraform.io/). It is highly recommended to look at the community-made modules before creating your own from scratch. The [State of DevOps](https://services.google.com/fh/files/misc/state-of-devops-2019.pdf) report shows that companies that re-use code are high performers.

In this lab, you will learn how to create reusable Terraform configurations using modules.

### Learning Objectives
Upon completion of this lab you will be able to:

- Understand how to use modules to create reusable infrastructure
- Know the benefits of using modules
- Learn how to pass data between modules
  
### Intended Audience
This lab is intended for:

Individuals studying to take the HashiCorp Certified: Terraform Associate exam
Anyone interested in learning how to use Terraform to manage Cloud Service Providers

### Lab Prerequisites
You should be familiar with:
- Cloud Services
- DevOps
- Deploying resources with Terraform and using variables

The following course can be used to fulfill the prerequisites:
- Introduction to DevOps
- Deploying Infrastructure with Terraform in Azure

Create a ```modules``` folder in your project directory
In the **modules** directory create:
- ```storage-account``` folder
In the in the **storage-account** folder create all the required Terraform files:

```main.tf```
```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
}

resource "azurerm_storage_account" "sa" {
  name                     = var.saname
  resource_group_name      = var.rgname
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = "GRS"

}
```

```variables.tf```
```hcl
variable "saname" {
    type = string
    description = "Name of storage account"
}
variable "rgname" {
    type = string
    description = "Name of resource group"
}
variable "location" {
    type = string
    description = "Azure location of storage account environment"
    default = "westus"
}
```

```output.tf```
```hcl
output "primary_key" {
    description = "The primary access key for the storage account"
    value = azurerm_storage_account.sa.primary_access_key
    sensitive   = true
}
```
Create a new ```main.tf``` outside  the project
```hcl
# Terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
}

#Azure provider
provider "azurerm" {
  features {}
}

#Create Storage Account
module "storage_account" {
  source    = "./modules/storage-account"

  saname    = "sacal32025671"
  rgname    = "cal-3202-567"
  location  = "westus"
}

#Create Storage Account
module "storage_account2" {
  source    = "./modules/storage-account"

  saname    = "sacal32025672"
  rgname    = "cal-3202-567"
  location  = "westus"
}
```
### Azure Run
```azure
$ az login
```

### Terraform Run

```hcl
$ terraform init

$ terraform plan

$ terraform apply

```

### Summary
In this lab step, you created a module for deploying Storage Accounts and called the module within a Terraform configuration file.
