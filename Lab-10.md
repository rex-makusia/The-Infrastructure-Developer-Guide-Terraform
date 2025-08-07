# Creating Loops in Terraform and Scaling Resources in Azure

## Description
Using loops in Terraform can be very powerful. It can take messy spaghetti code and turn it into something clean, simple, and easily understandable. It can make infrastructure scalable and efficient with just a few lines of code. The HCL language is both human-readable and machine-friendly, where components and lists can be iterated to create looping logic. Loops are commonly used in the Terraform community to make modules dynamic. For example, a Virtual Machine module can use loops to deploy multiple disks if needed or contain multiple IP addresses.

Using loops in Terraform can give code a cleaner look and follows the DRY (Don't Repeat Yourself)  principles of programming where the same concepts aren't repeated throughout the Terraform configuration. Loops also allow for resources to scale efficiently. 

In this lab, you will create a NSG Module that contains loops using several methods in Terraform.

You will create a module for deploying an Network Security Group. It is recommended to develop configurations with modules in mind. Local modules can be used to organize code. In this lab, the following folder hierarchy will be used:

Project structure
```yaml
terraformlab
    └──modules 
            └──nsg
                └── main.tf
                └── variables.tf
```

### Learning Objectives
Upon completion of this lab you will be able to:

- Understand how Terraform code can be made dynamic for reusability using loops
- Learn about count and for_each loops

### Intended Audience
This lab is intended for:

Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers

### Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform modules and variables

```variables.tf```
```hcl
variable "nsg_rule" {
    description = "OS image to deploy"
    type = list(object({
        name                       = string
        priority                   = number
        direction                  = string
        access                     = string
        protocol                   = string
        source_port_range          = string
        destination_port_range     = string
        source_address_prefix      = string
        destination_address_prefix = string
  }))
}    


variable "nsgname" {
    type = string
    description = "Name of NSG"
}
variable "rgname" {
    type = string
    description = "Name of resource group"
}
variable "location" {
    type = string
    description = "Azure location of NSG environment"
    default = "westus2"
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

# Network Security Group
resource "azurerm_network_security_group" "nsg" {
  name                = var.nsgname
  location            = var.location
  resource_group_name = var.rgname

  dynamic security_rule {
    for_each = var.nsg_rule
    content{
      name                       = security_rule.value.name
      priority                   = security_rule.value.priority
      direction                  = security_rule.value.direction
      access                     = security_rule.value.access
      protocol                   = security_rule.value.protocol
      source_port_range          = security_rule.value.source_port_range
      destination_port_range     = security_rule.value.destination_port_range
      source_address_prefix      = security_rule.value.source_address_prefix
      destination_address_prefix = security_rule.value.destination_address_prefix
    }
  }
}
```

terraformlab > maint.tf
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

# NSG module
module "nsg" {
    source = "./modules/nsg"

    nsgname    = "nsg"
    rgname    = "cal-3105-149"
    location  = "westus"

    nsg_rule =[
    {
    name                       = "http"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
    },
    {
     name                       = "ssh"
    priority                   = 101
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "22"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
    }
]
}
```
#### Terraform Run
```hcl
terraform init

terraform apply
```

terraformlab/main.tf
```maint.tf```
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

# NSG module
module "nsg" {
    count = 3 
    source = "./modules/nsg"

    nsgname    = "nsg${count.index}"
    rgname    = "cal-3105-149"
    location  = "westus"

    nsg_rule =[
    {
    name                       = "http"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "80"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
    },
    {
     name                       = "ssh"
    priority                   = 101
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "22"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
    }
]
}
```
The three NSGs are deployed with the Network Security Groups rules using the dynamic block. 

### Summary
In this lab step, you created a Network Security Group module that contains a dynamic block for security rules. You also used count to scale the module to deploy 3 NSGs.
