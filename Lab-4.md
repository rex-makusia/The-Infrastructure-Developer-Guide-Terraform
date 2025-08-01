# Using Provisioners with Terraform in Azure

# Description
Terraform uses HCL, a declarative language that is great for managing the abundance of cloud resources. 
However, it's declarative nature introduces pain points in automation tasks where procedural languages like PowerShell or Bash perform much better. 
Provisioners can help with this by automating additional steps when creating or destroying Terraform resources. 
They can be a powerful tool in the Terraform developer's toolbelt. 
However, it's important to know from the start that with provisioners comes a cost. 
Introducing provisioners into Terraform code can cause the code to become complex and introduce other pain points like network or software dependencies in the Terraform configuration. 
This is why it is important to be aware of how to use provisioners and understand that they should be used as a last resort when developing a solution.

In this lab, you will learn how to create a provisioner to automate additional tasks when creating and destroying an ACR resource.

# Learning Objectives
Upon completion of this lab you will be able to:

- Understand how to create Terraform provisioners
- Know when to use provisioners
  
# Intended Audience
This lab is intended for:

- Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers

# Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform deployments

The following course and lab can be used to fulfill the prerequisites:
- Introduction to DevOps
- Deploying Infrastructure with Terraform in Azure

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

provider "azurerm" {
  features {}
}


resource "azurerm_container_registry" "acr-dev" {
  name                     = "acrdevregistrycloudacademylab001"
  resource_group_name      = "cal-1035-3b1"
  location                 = "West US"
  sku                      = "Standard"
  admin_enabled            = false

  provisioner "local-exec" {
    when = destroy
    command = <<EOT
       az acr repository delete --name ${self.name} --image hello-world:calab --yes
    EOT
  }


}

#Import Container Image to Azure Container Registries
resource "null_resource" "image" {

  provisioner "local-exec" {
    command = <<EOT
       az acr import --name ${azurerm_container_registry.acr-dev.name} --source mcr.microsoft.com/hello-world --image hello-world:calab
    EOT
  }
}
```
# Terraform Run

```hcl
$ terraform init

$ terraform apply

$ terraform destroy
```
Keep in mind that provisioners are extremely useful for adding additional tasks to deploying and destroying resources. 
However, provisioners should be a last resource because it adds complexity to Terraform code and can make it brittle. 
It's best to seek alternative solutions before using provisioners. 
In this example, instead of pushing the docker image through a Terraform provisioner, it would be best to use a CI/CD tool like Azure DevOps or Jenkins to perform that task and let the Terraform code handle the infrastructure. 

# Summary
In this lab step, you created a provisioner to upload a Docker image to an ACR resource after it was provisioned. You also created a destroy provisioner to perform clean up tasks when the ACR was destroyed.
