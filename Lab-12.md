# Keeping Terraform Remote State DRY with Terragrunt in Azure

## ​Introduction
Terragrunt is a thin wrapper around Terraform that provides additional tools for keeping configurations DRY. 
DRY is a popular software development practice that stands for "Don't Repeat Yourself," which means don't reuse the same software pattern, avoiding redundancy. 
Terragrunt enables developers to use DRY principles when configuring remote state. For information on how to install Terragrunt, check out the 
[install documentation](https://terragrunt.gruntwork.io/docs/getting-started/install/#install-terragrunt)


In this lab step, you will use Terragrunt to deploy a Terraform configuration in separate environments while keeping the remote state configuration DRY.

## Instructions
1. Before you begin working with this lab, ensure that the provided environment and IDE development environment have fully provisioned:

2. Left-click on the Terminal menu and click New Terminal:

A Terminal window will appear at the bottom.

3. Inside the terminal, input the following command to authenticate with Azure:

```
az login
```

Copy the code in the last command's output and click the link to navigate to the login page.

Paste the code at the prompt and click **Next.**

Log in using the lab user credentials below:

Email: Enter:
Copy code

Temporary Access Pass: Enter: 
Copy code

At the Are you trying to sign in to Microsoft Azure CLI? prompt, click Continue to finish authenticating the Azure CLI.

Within the Terminal you should see the results below with a list of available subscriptions and other relevant information about the account. In this case, your account should have access to the subscription starting with the name CloudAcademyLabs Prod:

Tip: You can resize the terminal to see the whole response by grabbing the bar above the terminal and moving using your cursor up or down:

4. On the left-hand side, review the folder hierarchy and files:

Note: All necessary files for this lab have been pre-populated. However, some values like the resource group and storage account depend on the current Lab Azure Environment. 
To simplify this step, instead of manually replacing placeholders in each file, just run the following commands in the terminal to automatically populate the required values:

```
find "./terraformlab" -name "*.tf" | xargs sed -i "s/REPLACEME/
cal-3132-92c
/g"
find "./terraformlab" -name "*.hcl" | xargs sed -i "s/REPLACEME/
cal-3132-92c
/g"
find "./terraformlab" -name "*.hcl" | xargs sed -i "s/STORAGEACCOUNT/
sacalabscal313292c
/g"
```
Each directory represents three separate environments Dev, Prod, and QA. The ```main.tf``` files contain the configuration for deploying a Virtual Network and Subnet in each environment. The ```terragrunt.hcl``` files contain the configuration for Terragrunt and are required to exist in order to use the tool. 

Notice there is a terragrunt.hcl file at the root of the directory. This file will contain the remote state configuration settings which will be used in each subfolder. The remote state configurations will only be specified once at the root level and used throughout the sub folders, staying true to DRY principles. 

5. Click on the ```terragrunt.hcl``` in the root parent directory:

On the right-hand side the contents of ```terragrunt.hcl``` are displayed:

<img width="292" height="202" alt="image" src="https://github.com/user-attachments/assets/b2e615bd-e1ae-4b55-8e68-73542977d181" />

# Remote backend settings for all child directories
```
remote_state {
  backend = "azurerm"
  config = {
    resource_group_name  = "
cal-3132-92c
"
    storage_account_name = "
sacalabscal313292c
"
    container_name       = "calab" 
    key                  = "${path_relative_to_include()}/terraform.tfstate"
  }
}
```
The ```remote_state``` block contains the remote state configuration settings for storing the Terraform state in a Storage Account. The value of the ```key``` argument contains ```${path_relative_to_include()}``` which is another Terragrunt built-in function. This function includes the relative folder path where the Terraform configuration is located, which you will see later.  The ${path_relative_to_include()} function allows the key path to be automated for each directory.

6. Click on the terragrunt.hcl file in the Prod directory:

<img width="271" height="255" alt="image" src="https://github.com/user-attachments/assets/20ffc414-77fd-43d2-ac77-fa245d14b0fa" />

On the right-hand side, the contents of ```terragrunt.hcl``` are displayed:

```
# Include all settings from the root terragrunt.hcl file
include {
  path = find_in_parent_folders()
}
```
The ```include``` block tells Terragrunt to use the configuration from the directory specified in the path argument. In this case, the path argument contains the value of another Terragrunt built-in function, 
```find_in_parent_folders()```. This function tells Terragrunt to use the ```terragrunt.hcl``` file in the parent folder. 
So Terragrunt will use the remote state configuration settings from the ```terragrunt.hcl``` in the parent folder as if it were copied and pasted into the current directory in the Prod folder.

Each ```main.tf``` file in the child folders contains an empty azurerm backend block:

<img width="287" height="198" alt="image" src="https://github.com/user-attachments/assets/fe56d87f-24f6-4683-bb94-6723e6725bf0" />


This makes the remote state configuration much easier to maintain since all the configuration settings are maintained in a single file at the root of the directory.

7. Inside the terminal, input the following command to apply the Terraform configuration in all child folders with a single command: 

```
terragrunt run-all apply

```

Terragrunt will only apply the Terraform configurations in child folders that contain a ```terragrunt.hcl``` file. 

8. Input y to apply the configuration in each folder:

<img width="1950" height="734" alt="image" src="https://github.com/user-attachments/assets/42346f98-69b5-40b8-881f-1f66c3185306" />


A ```terraform init``` and ```terraform apply``` are executed against all child folders. This process can take 30 seconds or more to complete.

The ```backend-config``` parameter in **terragrunt.hcl** file sets the key path contains the folder name of each environment, this allows for each environment to have its own unique key path for the backend configuration.

The resources are deployed successfully:

The infrastructure has been deployed in three separate environments with just a single command and a single remote state configuration. 

9. Input the following command to destroy all resources that have been provisioned in each folder:

```
terragrunt run-all destroy
```

Notice: The 
```
terragrunt run-all destroy
```
 command should be used with caution. It should not be used against production environments and is more suited to ephemeral test or QA environments. 

10. Input y to confirm the destruction of each resource:

The resources in each environment are destroyed with a single command. For more information on the use cases and functionality of Terragrunt, check out the Terragrunt 
{documentation](https://terragrunt.gruntwork.io/docs/)

Summary
In this lab step, you used Terragrunt to configure the remote state in each environment with DRY principles.


terraform
-> dev
```main.tf```
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.40.0"
    }
  }
  backend "azurerm" {}
}

provider "azurerm" {
  features {}
}

#Create virtual network
resource "azurerm_virtual_network" "vnet1" {
  name                = "vnet-dev-westus-001"
  address_space       = ["10.0.0.0/16"]
  location            = "westus"
  resource_group_name = "REPLACEME"
}

# Create subnet
resource "azurerm_subnet" "subnet1" {
  name                 = "snet-dev-westus-001"
  resource_group_name  = "REPLACEME"
  virtual_network_name = azurerm_virtual_network.vnet1.name
  address_prefixes     = ["10.0.0.0/24"]
}
```
```terragrunt.hcl```


