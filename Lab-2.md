# Hands-on lab
## Creating Terraform Configurations with Implicit Dependencies in Azure

# Description
Terraform uses graph theory to plot relationships between resources. This mathematical process is used to determine the necessary steps to take for provisioning and destroying infrastructure. Understanding how to create resource dependencies within Terraform configurations is crucial for Terraform developers. It can prevent headaches in the future with timing issues and bugs in infrastructure deployments.

In this lab, you will learn how to create implicit dependencies between resources and experience firsthand what happens when Terraform resource dependencies are overlooked.

# Learning Objectives
Upon completion of this lab you will be able to:

- Understand how Terraform deploys resources in order
- Troubleshoot Terraform dependency issues

# Intended Audience
This lab is intended for:

- Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers
  
# Prerequisites
You should be familiar with:
- Cloud Services
- DevOps
Basic understanding of Terraform deployments
The following course and lab can be used to fulfill the prerequisites:

- Introduction to DevOps
- Deploying Infrastructure with Terraform in Azure

```main.tf ```

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
    name                = "vnet-dev-sc-001"
    address_space       = ["10.0.0.0/16","10.1.0.0/16"]
    location            = "southcentralus"
    resource_group_name = "cal-1085-470"
}

# Create subnet
resource "azurerm_subnet" "subnet" {
  name                 = azurerm_virtual_network.vnet.name
  resource_group_name  = "cal-1085-470"
  virtual_network_name = "vnet-dev-sc-001"
  address_prefixes       = ["10.0.0.0/24"]
}

## Terraform
```hcl
$ terraform init

$ terraform apply

$ terraform graph

# Before the hard-coded values

digraph {
        compound = "true"
        newrank = "true"
        subgraph "root" {
                "[root] azurerm_subnet.subnet (expand)" [label = "azurerm_subnet.subnet", shape = "box"]
                "[root] azurerm_virtual_network.vnet (expand)" [label = "azurerm_virtual_network.vnet", shape = "box"]
                "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"]" [label = "provider[\"registry.terraform.io/hashicorp/azurerm\"]", shape = "diamond"]
                "[root] azurerm_subnet.subnet (expand)" -> "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"]"
                "[root] azurerm_virtual_network.vnet (expand)" -> "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"]"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] azurerm_subnet.subnet (expand)"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] azurerm_virtual_network.vnet (expand)"
                "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"] (close)" -> "[root] azurerm_subnet.subnet (expand)"
                "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"] (close)" -> "[root] azurerm_virtual_network.vnet (expand)"
                "[root] root" -> "[root] meta.count-boundary (EachMode fixup)"
                "[root] root" -> "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"] (close)"
        }
}

# After fixing the vnet
digraph {
        compound = "true"
        newrank = "true"
        subgraph "root" {
                "[root] azurerm_subnet.subnet (expand)" [label = "azurerm_subnet.subnet", shape = "box"]
                "[root] azurerm_virtual_network.vnet (expand)" [label = "azurerm_virtual_network.vnet", shape = "box"]
                "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"]" [label = "provider[\"registry.terraform.io/hashicorp/azurerm\"]", shape = "diamond"]
                "[root] azurerm_subnet.subnet (expand)" -> "[root] azurerm_virtual_network.vnet (expand)"
                "[root] azurerm_virtual_network.vnet (expand)" -> "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"]"
                "[root] meta.count-boundary (EachMode fixup)" -> "[root] azurerm_subnet.subnet (expand)"
                "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"] (close)" -> "[root] azurerm_subnet.subnet (expand)"
                "[root] root" -> "[root] meta.count-boundary (EachMode fixup)"
                "[root] root" -> "[root] provider[\"registry.terraform.io/hashicorp/azurerm\"] (close)"
        }
}

Paste the output on http://www.webgraphviz.com/
```

## Azure login
```bash
$ az login
```
