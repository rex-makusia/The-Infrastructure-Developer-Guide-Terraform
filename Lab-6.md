# Using Terraform Remote State in Azure

## Description
Terraform state is a critical component of using Terraform. However, there are many risks involved when storing the state file locally on a computer. The state file can contain sensitive information about the infrastructure, such as keys and passwords. Saving them to an engineer's laptop is not a great long term solution. Also, as teams start utilizing Terraform and managing resources, they need a way to centrally store the state file so that everyone has access to run Terraform commands against the same infrastructure. Additionally, this centralized storage solution needs to have a system to prevent the same state file from being used simultaneously. This is why Terraform can be configured to use a remote state feature, where the state file is stored in a remote location, and a feature called state file locking is introduced. Remote state can be used with the native Azure Storage Account service.

In this lab, you will create a Storage Account and configure remote state for a Terraform Configuration.

### Learning Objectives
Upon completion of this lab you will be able to:
- Understand how Terraform remote state works
- Learn the benefits of remote state

### Intended Audience
This lab is intended for:

Individuals studying to take the HashiCorp Certified: Terraform Associate exam
Anyone interested in learning how to use Terraform to manage Cloud Service Providers


### Lab Prerequisites
You should be familiar with:
- Cloud Services
- DevOps
- Basic understanding of Terraform state
  
The following course and lab can be used to fulfill the prerequisites:

- Introduction to DevOps
- Exploring Terraform State in Azure
- Using the Azure CLI

#### Azure Commands
```azure
$ az login

$ az storage account create --name sasacal33282f988 --resource-group cal-3328-2f9 --location westus --sku Standard_LRS --encryption-services blob 

$ az storage container create --name calab --account-name sasacal33282f988 --auth-mode login

$ az storage account blob-service-properties update -n sasacal33282f988 --enable-change-feed true --enable-versioning true

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
   backend "azurerm" {
    resource_group_name  = "cal-3328-2f9"
    storage_account_name = "sasacal33282f988"
    container_name       = "calab"
    key                  = "dev.terraform.tfstate"
  }
}

#Azure provider
provider "azurerm" {
  features {}
}


#Create virtual network
resource "azurerm_virtual_network" "vnet" {
  name                = "vnet-dev-westus-001"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "cal-3328-2f9"
}

# Create subnet
resource "azurerm_subnet" "subnet" {
  name                 = "snet-dev-westus-001"
  resource_group_name  = "cal-3328-2f9"
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = ["10.0.0.0/24"]
}
```

#### Terraform Run
```hcl
$ cd terraformlab

$ terraform init

$ terraform plan

$ terraform apply
````

#### Azure
Verify the statefile
```azure
$ key=$(az storage account keys list -g cal-3328-2f9 -n sasacal33282f988 --query [0].value -o tsv)

az storage blob list --container-name calab --account-name sasacal33282f988 --account-key $key
```

# Summary
In this lab step, you created an Azure Storage Account and used it to configure remote state for a terraform configuration.
