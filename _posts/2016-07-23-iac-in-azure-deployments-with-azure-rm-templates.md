---
layout: post
title:  "IaC in Azure: Deployments with Azure RM Templates"
date:   2016-07-21 23:59:30 -0500
categories: azure iac
---

IaC in Azure: Deployments with Azure RM Templates
=================================================

*This is the first in a series of posts on tools and services to enable product teams to follow modern software delivery principles.*

While most software teams are storing their application’s code in source control, many teams are now storing the infrastructure code in source control, too. The rise of virtual machines allowed operations teams to easily and quickly provision new machines. Cloud computing services, such as Amazon Web Services and Azure, fundamentally changed the economics of data centers. Large datacenters can achieve economies of scale by building out massive data centers and renting out the extra capacity. Each physical server can run multiple virtual servers, thereby increasing the utilization of each server.

Infrastructure as Code
----------------------

One of the benefits of virtualization is automation. Provisioning servers in a cloud computing system can be completely automated. Automation is desirable because it lets you develop a repeatable provisioning process, decreasing the opportunity for user error. If the automated process is incorrect, rather than correcting the server, you correct the automation process and run it again. This helps ensure that the problem does not reappear.

A repeatable provisioning and deployment plan provides many benefits. One is that it forces documentation of infrastructure requirements. If it’s in the form of scripts or configuration files tracked in source control, it provides the additional benefit of being versioned. When rolling back to an older version of an application, any changes in infrastructure can be rolled back as well.

Infrastructure as code is the concept of storing information about provisioning of resources in code that is usually stored alongside the application code in source control. It could be some shell scripts or PowerShell scripts that spin up new virtual machines. For even a moderately large system however, this can lead to an unwieldly amount of scripting. Tools such as Chef and Puppet provide a more declarative approach. That is, rather than writing scripts that say what to do, you write scripts that describe what you want.

HashiCorp offers many tools, such as Packer and Vagrant, that allow infrastructure requirements to be stored in code. Vagrant is a tool for managing development environments. Packer is a tool for creating virtual machine images. These images can then be deployed to VMware, VirtualBox or AWS, among others.

While Packer can create a basic virtual machine image, your application will likely need some other software to be installed and configured on that machine. For example, a database server would need a database like MongoDB installed, and a web server might need Nginx. This is where a configuration management tool like Chef or Puppet comes in. These tools let you install software and define it in code.

You can use the tools I’ve covered so far to deploy a virtual machine in a number of different cloud providers, like Amazon, Digital Ocean, or Azure. Not every application is going to run on a virtual machine, however. Many applications are built to take advantage of resources like Web Apps, Azure SQL or Scheduler. Tools like Packer and Chef can provision virtual machines, but in order to provision a Web App or Azure SQL, we need something different.

Azure Resource Manager
----------------------

Azure provides many interfaces for automating provisioning, such as a REST API, command line tools, and API bindings for several programming languages. Any of these could be used to procedurally provision and configure resources in Azure. In addition to a facelift, part of the move to the new portal was a core change in the way resources are provisioned. The new portal uses the Azure Resource Manager to provision resources.

The Azure Resource Manager (ARM) uses JSON templates to declare resources and group them into a resource group. The ARM reads the template and provisions the resources accordingly. This template provisions a resource group with an Azure DocumentDB resource:

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "apiVersion": "2015-04-08",
                "type": "Microsoft.DocumentDb/databaseAccounts",
                "name": "to-do-list-db",
                "location": "eastus",
                "properties": {
                    "name": "to-do-list-db",
                    "databaseAccountOfferType": "Standard"
                }
            }
        ]
    }
    
To provision this, you can use the Azure CLI:

    azure login
    azure config mode arm
    azure group create --name ToDoListRG --location eastus --template-file azuredeploy.json
    
Templates can also have parameters and variables. We might want to pass in the resource name as a parameter, in which case we could rewrite the template:
    
    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "databaseAccountName": {
                "type": "string"
            }
        },
        "variables": {},
        "resources": [
            {
                "apiVersion": "2015-04-08",
                "type": "Microsoft.DocumentDb/databaseAccounts",
                "name": "[parameters('databaseAccountName')]",
                "location": "eastus",
                "properties": {
                    "name": "[parameters('databaseAccountName')]",
                    "databaseAccountOfferType": "Standard"
                }
            }
        ]
    }
    
If deploying this with the Azure CLI, the easiest way to supply the parameters is with a parameters file:

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "databaseAccountName": {
                "value": "to-do-list-db"
            }
        }
    }
    
To provision with the parameter file:

    azure group create --name ToDoListRG --location eastus --template-file azuredeploy.json --parameters-file azuredeploy.parameters.json
    
ARM Tooling for Visual Studio
-----------------------------

Authoring long JSON files by hand can get pretty tedious and messy. Fortunately, included in the [Azure SDK for .NET] (https://azure.microsoft.com/en-us/downloads/) is the Azure Resource Manager Tools for Visual Studio. With this installed, you can create an Azure Resource Group project.

*This post covers features in active development and will be updated accordingly.*

Further Reading
---------------

-   https://www.thoughtworks.com/insights/blog/infrastructure-code-reason-smile
-   https://azure.microsoft.com/en-us/documentation/articles/resource-group-overview/
-   https://azure.microsoft.com/en-us/blog/azure-resource-manager-2-5-for-visual-studio/
-   https://azure.microsoft.com/en-us/documentation/articles/automation-dsc-overview/
-   http://martinfowler.com/bliki/InfrastructureAsCode.html

References
----------

-   Limoncelli, Thomas A, Strata R Chalop, Christina J Hogan, *Practice of Cloud System Administration*
-   Humble, Jez, David Farley, *Continuous Delivery*
