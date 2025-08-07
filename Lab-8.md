# Importing Existing Infrastructure into Terraform in Azure

## Description
Terraform is a great way to deploy and manage infrastructure. But what about the existing infrastructure that has been deployed and managed manually? 
One of the main principles of infrastructure development is to "define everything in code'. To fully reap the benefits of operating your infrastructure through Terraform, 
pre-existing infrastructure should be managed and used in the same manner to prevent one-off environments and reduce risk. This often takes time and effort and is usually a considerable-sized 
project to migrate old infrastructure into Terraform retroactively.

### Terraform Import Workflow
<img width="767" height="142" alt="image" src="https://github.com/user-attachments/assets/be19d118-c9ac-43d6-bd5e-4fe52a35a924" />

In this lab, you will import pre-existing infrastructure into a Terraform state file managed by Terraform.

###  Learning Objectives
Upon completion of this lab you will be able to:

- Understand how existing Azure resources are mapped to a Terraform state file
- Learn the import process for most Terraform resources
  
### Intended Audience
This lab is intended for:

- Individuals studying to take the HashiCorp Certified: Terraform Associate exam
- Anyone interested in learning how to use Terraform to manage Cloud Service Providers
  
### Lab Prerequisites
You should be familiar with:

- Cloud Services
- DevOps
- Basic understanding of Terraform state
  
The following course and lab can be used to fulfill the prerequisites:
- Introduction to DevOps
- Exploring Terraform State in Azure

#### Terraform Configuration

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

resource "azurerm_virtual_network" "vnet" {}
```

#### Azure Run
```az
az login

bash-5.0$ az login
To sign in, use a web browser to open the page https://microsoft.com/devicelogin and enter the code AV9JV9KMD to authenticate.
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "fd1fbf9f-991a-40b4-ae26-61dfc34421ef",
    "id": "b197ebe7-8059-4dbe-90a8-15f7bf3d746f",
    "isDefault": true,
    "managedByTenants": [],
    "name": "CloudAcademyLabs Prod 4",
    "state": "Enabled",
    "tenantId": "fd1fbf9f-991a-40b4-ae26-61dfc34421ef",
    "user": {
      "name": "qa-student-965-po165h@azure.labs.platform.qa.com",
      "type": "user"
    }
  }
]
bash-5.0$ 
```

#### Terraform Run
```hcl
terraform init
```

Get the resource ID
```az
VNETID=$(az network vnet show -g cal-965-52d -n lab-vnet --query [id] --output tsv)

echo $VNETID

/subscriptions/b197ebe7-8059-4dbe-90a8-15f7bf3d746f/resourceGroups/cal-965-52d/providers/Microsoft.Network/virtualNetworks/lab-vnet
bash-5.0$ 

terraform import azurerm_virtual_network.vnet $VNETID

# azurerm_virtual_network.vnet:
resource "azurerm_virtual_network" "vnet" {
    address_space         = [
        "10.0.0.0/20",
    ]
    dns_servers           = []
    guid                  = "15543830-89ea-430b-8192-d387dd4c4948"
    id                    = "/subscriptions/b197ebe7-8059-4dbe-90a8-15f7bf3d746f/resourceGroups/cal-965-52d/providers/Microsoft.Network/virtualNetworks/lab-vnet"
    location              = "westus"
    name                  = "lab-vnet"
    resource_group_name   = "cal-965-52d"
    subnet                = [
        {
            address_prefix = "10.0.0.0/24"
            id             = "/subscriptions/b197ebe7-8059-4dbe-90a8-15f7bf3d746f/resourceGroups/cal-965-52d/providers/Microsoft.Network/virtualNetworks/lab-vnet/subnets/default"
            name           = "default"
            security_group = ""
        },
    ]
    tags                  = {}
    vm_protection_enabled = false

    timeouts {}
}

terraform show
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


resource "azurerm_virtual_network" "vnet" {
    address_space         = [
        "10.0.0.0/20",
    ]
    dns_servers           = []
    location              = "West US"
    name                  = "lab-vnet"
    resource_group_name   = "cal-965-52d"
    
    vm_protection_enabled = false

}
```
Run Terraform Plan
```hcl
terraform plan
```

It is important to understand how each Azure resource is imported into Terraform and highly advised to test before importing production infrastructure. You should not rely on the provider documentation to determine the outcome of the import. 

Importing existing infrastructure into Terraform can be a tedious process. However, the benefits of managing pre-existing infrastructure with Terraform are great and worth the effort. HashiCorp is currently in the process of improving the import experience and will be making efforts in the future to speed up the import process. You may also want to check out the community-made tool, 
[Terraformer](https://github.com/GoogleCloudPlatform/terraformer)
, opens in a new tab, that automates the Terraform import process for the major cloud providers. 

#### Summary
In this lab step, you successfully imported an Azure Virtual Network into a Terraform configuration that can now be managed with Terraform.
