# Description

Terraform is an infrastructure automation tool that allows companies to manage infrastructure through code. This provides many benefits such as greater recovery, predictability, and speed. Infrastructure as code is quickly becoming a standard for managing cloud resources and is widely practiced among the top high performing companies. One of the major benefits of using Terraform over other infrastructure as code tools is that its platform agnostic. Terraform can be used to manage any major Cloud Service Provider as well as on-prem environments using vSphere and Cisco. Teams can use the same adopted skillset and workflows to manage other environments instead of reinventing the wheel for managing each environment. Terraform is also simple to learn in a matter of a few days and can be quickly adopted as a way to manage infrastructure among teams.

In this lab, you will create a Terraform configuration to deploy a Virtual Network in Azure.

# Learning Objectives

Upon completion of this lab you will be able to:

- Understand what a Terraform provider is
- Use Terraform init, plan, and apply commands to build infrastructure
- Be familiar with the Hashicorp Configuration Language syntax and Terraform configuration files
- Build and destroy a Virtual Network in Azure

# Intended Audience
This lab is intended for:

- Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers

# Lab Prerequisites
You should be familiar with:

- Basic concepts of Cloud Service Providers

# HANDS-ON LAB
## Deploying Infrastructure with Terraform in Azure

## Introduction
Terraform uses configuration files ending in .tf to deploy and manage infrastructure. These configuration files are written in HashiCorp Configuration Language (HCL), designed to be both very human-readable and machine friendly. HCL serves the purpose of managing infrastructure, which means it must be simple enough to be "living documentation" of a system or environment. The  components created  in  HCL are described in blocks similar to JSON:

```hcl
<BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
  # Block body
  <IDENTIFIER> = <EXPRESSION> # Argument
}
```

```main.tf```

```hcl
terraform {
  required_providers {
    azure = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
}

provider "azurerm" {
  features {}
}

resource "azurerm_virtual_network" "vnet" {
  name                = "calabs-vnet"
  location            = "West US"
  resource_group_name = "cal-3198-be6"
  address_space       = ["10.0.0.0/16"]
}
```

## Terraform Deployment

```bash
$ terraform init

$ terraform plan

$ terraform apply

$ terraform destroy
```
