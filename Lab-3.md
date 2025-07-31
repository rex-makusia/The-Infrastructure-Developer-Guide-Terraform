# Creating Variables in Terraform Configuration in Azure

## Description
Variables are a critical component of developing Terraform code. They allow configurations to become dynamic and reusable by enabling them to take input values during deployment. By incorporating variables into your Terraform code, the same code used to deploy a Virtual Machine into an environment can be used to deploy into another. This strategy can improve the speed and efficiency of teams. For example, a database server takes a DB expert to install the server and configure the database to the most optimal settings. With Terraform, a database server build can be defined in code and use variables to configure the server's unique values for each environment. Other team members can then use the code to deploy the same configuration without the database expertise. 

In this lab, you will learn how to create reusable Terraform configurations using variables. 

## Learning Objectives
Upon completion of this lab you will be able to:

Understand how to create Terraform variables
Know the benefits of using variables

## Intended Audience
This lab is intended for:

Individuals studying to take the HashiCorp Certified: Terraform Associate exam
Anyone interested in learning how to use Terraform to manage Cloud Service Providers

## Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform deployments
  
The following course and lab can be used to fulfill the prerequisites:
- Introduction to DevOps
- Deploying Infrastructure with Terraform in Azure


# ​Introduction

Variables are a core part of Terraform and a heavily reutilized in most Terraform configurations. Hard coding values into Terraform configurations is discouraged if they are preventing the code from becoming a reusable tool.

```variables.tf```
```hcl
variable "location" {
    type = string
    description = "Azure location of terraform server environment"
    default = "westus2"

}

variable "vnet_address_space" { 
    type = list
    description = "Address space for Virtual Network"
    default = ["10.0.0.0/16"]
}

variable "snet_address_space" { 
    type = list
    description = "Address space for Virtual Network"
    default = ["10.0.0.0/24"]
}

variable "servername" {
    type = string
    description = "Server name of the virtual machine"
}

variable "admin_username" {
    type = string
    description = "Administrator username for server"
}

variable "admin_password" {
    type = string
    description = "Administrator password for server"
}


variable "managed_disk_type" { 
    type = map
    description = "Disk type Premium in Primary location Standard in DR location"

    default = {
        westus2 = "Premium_LRS"
        southcentralus = "Standard_LRS"
    }
}

variable "vm_size" {
    type = string
    description = "Size of VM"
    default = "Standard_B1s"
}

variable "os" {
    description = "OS image to deploy"
    type = object({
        publisher = string
        offer = string
        sku = string
        version = string
  })
}      
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

#Azure provider
provider "azurerm" {
  features {}
}


#Create virtual network
resource "azurerm_virtual_network" "vnet" {
    name                = "vnet-dev-${var.location}-001"
    address_space       = var.vnet_address_space
    location            = var.location
    resource_group_name = "cal-3415-9a1"
}

# Create subnet
resource "azurerm_subnet" "subnet" {
  name                 = "snet-dev-${var.location}-001 "
  resource_group_name  = "cal-3415-9a1"
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes       = var.snet_address_space
}


# Create network interface
resource "azurerm_network_interface" "nic" {
  name                      = "nic-01-${var.servername}-dev-001 "
  location                  = var.location
  resource_group_name       = "cal-3415-9a1"

  ip_configuration {
    name                          = "niccfg-${var.servername}"
    subnet_id                     = azurerm_subnet.subnet.id
    private_ip_address_allocation = "dynamic"
  }
}

# Create virtual machine
resource "azurerm_virtual_machine" "vm" {
  name                  = var.servername
  location              = var.location
  resource_group_name   = "cal-3415-9a1"
  network_interface_ids = [azurerm_network_interface.nic.id]
  vm_size               = "Standard_B1s"

  storage_os_disk {
    name              = "stvm${var.servername}os"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = lookup(var.managed_disk_type, var.location, "Standard_LRS")
  }

  storage_image_reference {
    publisher = var.os.publisher
    offer     = var.os.offer
    sku       = var.os.sku
    version   = var.os.version
  }

  os_profile {
    computer_name  = var.servername
    admin_username = var.admin_username
    admin_password = var.admin_password
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }
}
```

```terraform.tfvars```
```hcl
servername = "vmterraform"
location = "southcentralus"
admin_username = "terraadmin"
admin_password = "P@SSw0rdP@ssw0rd"
vnet_address_space = ["10.0.0.0/16"]
os = {
    publisher = "Canonical"
    offer     = "UbuntuServer"
    sku       = "16.04.0-LTS"
    version   = "latest"
}
```
Terraform will automatically look for a terraform.tfvars in the directory and use the values in that file for the variables.

This is a great way to avoid having to use -var for each value. However, be aware that sensitive variable information should only be used with -var and not written to theterraform.tfvars file.

```output.tf```
```hcl
output "private_ip" {
    description = "IP Address of Virtual Machine"
    value = azurerm_network_interface.nic.private_ip_address
}
```

## Azure Commands
```azure
$ az login

```

## Terraform Commands
```hcl
$ terraform init

$ terraform plan

$ terraform apply
terraform apply -var='vnet_address_space=["10.0.0.0/16"]' -var='snet_address_space=["10.0.0.0/24"]' -var="location=southcentralus"
```


## Summary
In this lab step, you used variables to design a Terraform configuration for deploying a Virtual Network, Subnet, and Virtual Machine. 
