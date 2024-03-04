---
layout: post
title:  "Deploy Bicep templates via Azure DevOps pipelines using secure variables"
date:   2024-03-04 20:20:00 +0100
categories: Bicep DevOps
---

So you want to deploy infrastructure using Bicep templates but don’t want to hard-code usernames and passwords within the template or in a cleartext parameters file. This is perfectly reasonable and very do-able.

In this walkthrough I’ll be using a Bicep template to deploy some infra including a VM for which I will need to specify a local admin username and password.

The Bicep template I’ll be using is from:

[https://learn.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-bicep?tabs=CLI](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/quick-create-bicep?tabs=CLI){:target="_blank"}  

The pipeline deployment code is from:

[https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/add-template-to-azure-pipelines?tabs=CLI
](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/add-template-to-azure-pipelines?tabs=CLI
){:target="_blank"} 

This walkthrough assumes you have set up and configured your DevOps organisation, project and service connection.

### The Repo

I created two files in the ‘BicepDeployWalkthrough’ project repo:

    Deploy_VM.bicep    
    azure-pipeline.yml

and pasted in the code from the templates at the above links, using the ‘Azure CLI task’ code for the YAML file.

I created a pipeline, ‘BicepDeployWalkthroughPipeline’ that uses the azure-pipeline.yml file, the code of which is:

    trigger:
    - main
    
    name: Deploy Bicep files
    
    variables:
      vmImageName: 'ubuntu-latest'
    
      azureServiceConnection: 'BicepDeployWalkthrough'
      resourceGroupName: 'Bicep-Deploy-RG'
      location: 'westeurope'
      templateFile: 'Deploy_VM.bicep'
    pool:
      vmImage: $(vmImageName)
    
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: $(azureServiceConnection)
        scriptType: bash
        scriptLocation: inlineScript
        useGlobalConfig: false
        inlineScript: |
          az --version
          az group create --name $(resourceGroupName) --location $(location)
          az deployment group create --resource-group $(resourceGroupName) --template-file $(templateFile)


Running this results in an error due to missing ‘adminUsername’ and ‘adminPassword’ values:

![](/assets/images/Bicep_Deploy/Pipeline_Error.png){:class="img-responsive"}{:width="80%"}

To fix this, browse to the pipeline -> Edit
Select ‘Variables’ and add two, one called adminUN and the other called adminPASS

If required, select ‘Keep this value secret’ and allow users to override the value.

Now you can add a '--parameters' switch to add the variables to the yml file.

azure-pipeline.yml now reads:

    trigger:
    - main
    
    name: Deploy Bicep files
    
    variables:
      vmImageName: 'ubuntu-latest'
    
      azureServiceConnection: 'BicepDeployWalkthrough'
      resourceGroupName: 'Bicep-Deploy-RG'
      location: 'westeurope'
      templateFile: 'Deploy_VM.bicep'
    pool:
      vmImage: $(vmImageName)
    
    steps:
    - task: AzureCLI@2
      inputs:
        azureSubscription: $(azureServiceConnection)
        scriptType: bash
        scriptLocation: inlineScript
        useGlobalConfig: false
        inlineScript: |
          az --version
          az group create --name $(resourceGroupName) --location $(location)
          az deployment group create --resource-group $(resourceGroupName) --template-file $(templateFile) --parameters adminUsername="$(adminUN)" adminPassword="$(adminPASS)"

Now the deployment will complete without needing any sensitive information in the template.