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
