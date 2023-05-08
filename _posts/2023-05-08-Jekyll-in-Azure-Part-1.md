---
layout: post
title:  "Jekyll in Azure - Part 1"
date:   2023-05-07 19:45:00 +0000
categories: Jekyll CloudResumeChallenge HowTo
---

This is the first in a series of posts around deploying a Jekyll site to Azure storage.  

The end goal is to be able to edit the site locally, sync changes to an Azure DevOps repository and have CI/CD pipelines generate the static site and publish to Azure storage. This tutorial comes in two parts. Part two will cover configuring auto-renewing LetsEncrypt SSL certs, Azure DNS, and Azure CDN.

Pre-requisites:

A Microsoft Azure subscription - [https://azure.microsoft.com/en-gb/free/](https://azure.microsoft.com/en-gb/free/){:target="_blank"}  
Jekyll theme – I am using PlainWhite: [https://github.com/samarsault/plainwhite-jekyll](https://github.com/samarsault/plainwhite-jekyll){:target="_blank"}  
VSCode - [https://code.visualstudio.com/](https://code.visualstudio.com/){:target="_blank"}  
Git - [https://git-scm.com/downloads](https://git-scm.com/downloads){:target="_blank"}  

The first task is to create a storage account and enable the static website feature.

Log in to the Azure portal and create a storage account:

![](/assets/images/Jekyll_in_Azure_Pt1/image001.png){:class="img-responsive"}{:width="50%"}

Once created, browse to the storage account -> Data management -> Static website -> Select “Enabled”, add Index and Error documents then save. This creates a container named $web in the storage account that will store site files and folders.

![](/assets/images/Jekyll_in_Azure_Pt1/image003.png){:class="img-responsive"}{:width="50%"}

Make a note of the URL for the site, listed as the Primary Endpoint:

![](/assets/images/Jekyll_in_Azure_Pt1/image005.png){:class="img-responsive"}{:width="50%"}

Search in the Azure Portal for “Devops”, create an organisation (if required), then create a project:

![](/assets/images/Jekyll_in_Azure_Pt1/image007.png){:class="img-responsive"}{:width="50%"}  

![](/assets/images/Jekyll_in_Azure_Pt1/image009.png){:class="img-responsive"}{:width="50%"}  

If this is a new organisation, request a free grant of parallel jobs from MS using the [AZPipelines Parallelism Request](https://aka.ms/azpipelines-parallelism-request){:target="_blank"} form.

Browse to the project repository -> Import a repository -> Import

I am using the Plainwhite theme available at: [https://github.com/samarsault/plainwhite-jekyll](https://github.com/samarsault/plainwhite-jekyll){:target="_blank"}

![](/assets/images/Jekyll_in_Azure_Pt1/image011.png){:class="img-responsive"}{:width="50%"}  

In the repository, click on “Clone” then “Generate Git Credentials” and make a note of the username and password.

![](/assets/images/Jekyll_in_Azure_Pt1/image013.png){:class="img-responsive"}{:width="50%"}  

Click on Open and select a location to store the code. Supply the generated password when prompted in VSCode.

![](/assets/images/Jekyll_in_Azure_Pt1/image015.png){:class="img-responsive"}{:width="50%"}  

You should now see your file structure replicated in VSCode and be able to make changes.

![](/assets/images/Jekyll_in_Azure_Pt1/image017.png){:class="img-responsive"}{:width="50%"}  

The first change is to add a file to the root of the folder called:

build.yml

and paste in the following:

    # Ruby
    # Package your Ruby project.
    # Add steps that install rails, analyze code, save build artifacts, deploy, and more:
    # https://docs.microsoft.com/azure/devops/pipelines/languages/ruby
    
    trigger:
    - master
    
    pool:
      vmImage: 'ubuntu-latest'
    
    steps:
    - task: UseRubyVersion@0
      inputs:
        versionSpec: '>= 2.5'
    
    - script: |
        gem install jekyll bundler
        bundle install --retry=3 --jobs=4
      displayName: 'bundle install'
     
    - script: |
        bundle install
        jekyll build
      displayName: 'jekyll'
     
    - task: CopyFiles@2
      displayName: 'Copy Files to: $(Build.ArtifactStagingDirectory)'
      inputs:
        SourceFolder: '_site'
        TargetFolder: '$(Build.ArtifactStagingDirectory)'
    
    - task: PublishBuildArtifacts@1
      inputs:
        pathtoPublish: '$(Build.ArtifactStagingDirectory)'
        artifactName: site
    
This file will control the build pipeline. It specifies the VM agent type to use, installs the required software, builds the Jekyll site and outputs it as an artifact.

Next browse to Source Control -> “Commit” -> enter a commit message -> click on the tick at top right to accept:

![](/assets/images/Jekyll_in_Azure_Pt1/image019.png){:class="img-responsive"}{:width="100%"}  

Now sync changes:

![](/assets/images/Jekyll_in_Azure_Pt1/image021.png){:class="img-responsive"}{:width="50%"}  

Your DevOps repository in Azure should now contain the build.yml file.

![](/assets/images/Jekyll_in_Azure_Pt1/image023.png){:class="img-responsive"}{:width="50%"}  

## Set up the Build pipeline

In your Azure DevOps project, browse to “Pipelines”, click on “Create Pipeline” and select “Azure Repos Git”

![](/assets/images/Jekyll_in_Azure_Pt1/image025.png){:class="img-responsive"}{:width="50%"}  

Select “Existing Azure Pipelines YAML file”

![](/assets/images/Jekyll_in_Azure_Pt1/image027.png){:class="img-responsive"}{:width="50%"}  

Select the build.yml file created earlier:

![](/assets/images/Jekyll_in_Azure_Pt1/image029.png){:class="img-responsive"}{:width="50%"}  

Run the pipeline and it will generate the complete site files and output them as an artifact.

If you get a “No hosted parallelism has been purchased or granted” error, request a free grant of parallel jobs from MS using the form linked above.

![](/assets/images/Jekyll_in_Azure_Pt1/image031.png){:class="img-responsive"}{:width="75%"}  

You will be notified by email once you have received the grant – it may be a while but normally completes within 24 hours.

## Set up the Deploy pipeline

Once the build pipeline has run successfully, we can create a deployment pipeline to push the artifact to the Azure Storage $web container.

Browse to Pipelines -> Releases -> click on “New pipeline” and select “Empty job”:

![](/assets/images/Jekyll_in_Azure_Pt1/image033.png){:class="img-responsive"}{:width="50%"}  

In “Artifacts”, click on “Add” and select the build from the previous pipeline:

![](/assets/images/Jekyll_in_Azure_Pt1/image035.png){:class="img-responsive"}{:width="30%"}  

Click on “Add”.

![](/assets/images/Jekyll_in_Azure_Pt1/image037.png){:class="img-responsive"}{:width="50%"}  

In order to have the release pipeline trigger after every new build:

Click on the lightning icon:

![](/assets/images/Jekyll_in_Azure_Pt1/image039.png){:class="img-responsive"}{:width="30%"}  

Switch the slider to Enabled and add the master branch to the build branch filters:

![](/assets/images/Jekyll_in_Azure_Pt1/image041.png){:class="img-responsive"}{:width="50%"}  

In Stages, click on the “1 job, 0 task” link

![](/assets/images/Jekyll_in_Azure_Pt1/image043.png){:class="img-responsive"}{:width="50%"}  

This will open the Deployment process Agent job.

Add a command line task to the agent job:

![](/assets/images/Jekyll_in_Azure_Pt1/image045.png){:class="img-responsive"}{:width="75%"}  

In the task, add to the script:

    del *.yml  
    del *.gemspec  

and set the working directory to:

    $(System.DefaultWorkingDirectory)/_*Your_ADO_Project_Name*/site

![](/assets/images/Jekyll_in_Azure_Pt1/image047.png){:class="img-responsive"}{:width="50%"}  

Second, add an “Azure CLI” task:

![](/assets/images/Jekyll_in_Azure_Pt1/image049.png){:class="img-responsive"}{:width="50%"}  

ADO needs a means to upload the site into the storage account. It uses a Service Principal to do this. To create a Service Principal, in the Azure CLI task, click on “Manage”:

![](/assets/images/Jekyll_in_Azure_Pt1/image051.png){:class="img-responsive"}{:width="50%"}  

Click on “Create service connection” and select “Azure Resource Manager”, then "Service principal (automatic)":

![](/assets/images/Jekyll_in_Azure_Pt1/image053.png){:class="img-responsive"}{:width="50%"}  

![](/assets/images/Jekyll_in_Azure_Pt1/image055.png){:class="img-responsive"}{:width="50%"}  

![](/assets/images/Jekyll_in_Azure_Pt1/image057.png){:class="img-responsive"}{:width="50%"}  

Select the storage account’s subscription and resource group, give the connection a name, and tick “Grant access permission to all pipelines”:

![](/assets/images/Jekyll_in_Azure_Pt1/image059.png){:class="img-responsive"}{:width="50%"}  

Save and wait for the process to complete.

Back in the Azure CLI task, select the newly created Service Principal:

![](/assets/images/Jekyll_in_Azure_Pt1/image061.png){:class="img-responsive"}{:width="50%"}  

Configure the options as follows:

    Script type:			Batch  
    Script location:		Inline script  
    Inline script:			az storage blob sync -s site -c $web  
      
    Working Directory:		$(System.DefaultWorkingDirectory)/_Your_ADO_Project_Name
      
    Environment Variables:	AZURE_STORAGE_ACCOUNT

Set the AZURE_STORAGE_ACCOUNT value to the name of your storage account.

![](/assets/images/Jekyll_in_Azure_Pt1/image063.png){:class="img-responsive"}{:width="50%"}  

Click on the Save icon at the top of the page -> OK

Next click on “Create release”:

![](/assets/images/Jekyll_in_Azure_Pt1/image065.png){:class="img-responsive"}{:width="50%"}  

Click “Create”

![](/assets/images/Jekyll_in_Azure_Pt1/image067.png){:class="img-responsive"}{:width="50%"}  

Click on the “Release-1” link to open the release pipeline:

![](/assets/images/Jekyll_in_Azure_Pt1/image069.png){:class="img-responsive"}{:width="50%"}  

Once the Stage 1 has been queued you can click on it and check its progress 

![](/assets/images/Jekyll_in_Azure_Pt1/image071.png){:class="img-responsive"}{:width="50%"}  

You can view the logs in real-time:

![](/assets/images/Jekyll_in_Azure_Pt1/image073.png){:class="img-responsive"}{:width="50%"}  

Once the job has completed, check that the site has deployed correctly to the storage account URL:

You should get the default PlainWhite page:

![](/assets/images/Jekyll_in_Azure_Pt1/image075.png){:class="img-responsive"}{:width="50%"}  

Now you have the site up and running, you can start to make changes. For quick visual feedback you can serve the site locally. In VSCode, open a terminal and run: 

    bundle exec Jekyll serve

This will serve the site on http://127.0.0.1:4000

You can quickly see the effect of changes and only trigger a Build/Deploy when you're ready.

For information on customising your site, have a look at the [Jekyll docs](https://jekyllrb.com/docs/configuration/ ){:target="_blank"}.

The next in this series of posts, which covers Azure DNS, Azure CDN, and auto-renewing SSL certs, can be found [here]({% post_url 2023-05-07-Jekyll-in-Azure-Part-2 %}){:target="_blank"}.