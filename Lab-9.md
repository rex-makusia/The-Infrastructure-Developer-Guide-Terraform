# Integrating Conditional Logic into Terraform Configurations in Azure

## Description
As companies start to build infrastructure using Terraform, the logic required to build self-service automation becomes complex. That's why HCL (HashiCorp Configuration Language) contains expressions that can perform calculations within Terraform configurations. Integrating conditional logic into modules allows them to become dynamic and reusable for different scenarios. 

In this lab, you will create a Storage Account Module that contains conditional logic for deploying a Storage Account. 

### Introduction

Creating conditional logic within Terraform configurations allows for creating a more dynamic configuration. This enables greater abstraction and logic flow in the code. 
It's essential to understand the logical conditions that can be created within the HCL (HashiCorp Configuration Lanuage) language as it provides even more reusability.

In this lab step, you will create a module with conditional logic to make it more dynamic.

### Learning Objectives
Upon completion of this lab you will be able to:

Understand how Terraform modules can be made dynamic for reusability
Learn about using conditional expressions within Terraform configurations

### Intended Audience
This lab is intended for:

Individuals studying to take the HashiCorp Certified: Terraform Associate exam
Anyone interested in learning how to use Terraform to manage Cloud Service Providers

### Lab Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform modules and variables
  
The following course and lab can be used to fulfill the prerequisites:

- Introduction to DevOps
- Creating Reusable Infrastructure with Terraform Modules in Azure

### Modules Creation
```yaml
terraform 
  -> modules
          -> storage-account
              -> maint.tf
              -> variables.tf
```

```main.tf```
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

#Storage Account
resource "azurerm_storage_account" "sa" {
  name                     = var.saname
  resource_group_name      = var.rgname
  location                 = var.location != "" ? var.location : "southcentralus"
  account_tier             = "Standard"
  account_replication_type = "GRS"
}
```
Types of [equality operators](https://developer.hashicorp.com/terraform/language/expressions#arithmetic-and-logical-operators)

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
    default = ""
}
```
terraform > main.tf

```main.tf```
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

  saname    = "sacal3484a761"
  rgname    = "cal-3484-a76"
}

#Create Storage Account
module "storage_account2" {
  source    = "./modules/storage-account"

  saname    = "sacal3484a762"
  rgname    = "cal-3484-a76"
  location  = "westus"
}
```

#### Terraform Run
```hcl
terraform init

terraform apply

```

The Storage Account modules have been deployed successfully in different locations. The conditional logic created in the Terraform module makes it dynamic and allows it to be used in several different scenarios. 

#### Summary
In this lab step, you created a Storage Account module that uses a conditional expression to optionally take location input to deploy an Azure Storage Account.
