# Exploring Terraform State in Azure

## Description
Terraform uses a system called Terraform State to keep track of resources managed and deployed through Terraform. This is a critical concept to understand as a Terraform infrastructure developer because Terraform state will always need to be kept in mind when architecting solutions. The state file acts as a database for mapping resources in Azure to the Terraform configuration file. It allows Terraform to have a declarative approach when it comes to infrastructure and provides performance boosts with large scale infrastructure automation. 

In this lab, you will learn how Terraform keeps track of the resources that it manages through Terraform state.

## Learning Objectives
Upon completion of this lab you will be able to:
- Understand how Terraform state works
- Learn the security risks of the state file
  
## Intended Audience
This lab is intended for:

- Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers
  
## Lab Prerequisites
You should be familiar with:

- Cloud Services
- DevOps

The following course can be used to fulfill the prerequisites:

- Introduction to DevOps

```main```
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
#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet1-dev-westus-001"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "cal-3280-b45"
}
# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet1-dev-westus-001"
  resource_group_name  = "cal-3280-b45"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  # address_prefixes     = ["10.0.0.0/24"] Update the address_prefix to below to see the changes on the terraform.tfstate file
  address_prefixes       = ["10.0.3.0/24"]
}
#Create virtual network
resource "azurerm_virtual_network" "vnet2" {
  name                = "vnet2-dev-westus-001"
  address_space       = ["10.1.0.0/16"]
  location            = "westus"
  resource_group_name = "cal-3280-b45"
}
# Create subnet
resource "azurerm_subnet" "subnet2" {
  name                 = "snet2-dev-westus-001"
  resource_group_name  = "cal-3280-b45"
  virtual_network_name = azurerm_virtual_network.vnet2.name
  #address_prefixes     = ["10.1.0.0/24"]
}
```
## Azure Command
```az
az login
```

## Terraform
```hcl
$ terraform init

$ terraform apply
```

# Bash
```sh
$ cd terraformlab
$ rm terraform.tfstate
```

## Terraform
```hcl
$ terraform apply
```

# Bash
```sh
cp terraform.tfstate.backup terraform.tfstate
```
# Remove Subnet2
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
#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet1-dev-westus-001"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "cal-3280-b45"
}
# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet1-dev-westus-001"
  resource_group_name  = "cal-3280-b45"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  address_prefixes     = ["10.0.3.0/24"]
}
#Create virtual network
resource "azurerm_virtual_network" "vnet2" {
  name                = "vnet2-dev-westus-001"
  address_space       = ["10.1.0.0/16"]
  location            = "westus"
  resource_group_name = "cal-3280-b45"
}
```

## Terraform
```hcl
$ terraform apply
```

The key takeaways to remember:

- Terraform is declarative, infrastructure is described in the configuration file, and Terraform handles the automation and provisioning of those resources.  
- The Terraform configuration file represents the current infrastructure. It will become unusable if the configuration becomes inaccurate due to manually modifying the Azure resources instead of using Terraform.
- Store the state file in a restricted location because it can contain sensitive information like passwords and keys.

## Summary
In this lab step, you explored Terraform state and how Terraform uses it to keep track of resources.
