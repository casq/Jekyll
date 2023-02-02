---
layout: post
title:  "Jekyll in Azure Blob Storage with Azure DevOps CI/CD"
date:   2023-01-12 00:03:36 +0530
categories: Jekyll GettingStarted
---
Initial notes

The goal of this project is to deploy a Jekyll website to Azure blob storage using Azure DevOps CI/CD, and be able to edit the site locally using VSCode

VSCode is available here:

<a href="https://code.visualstudio.com/Download">https://code.visualstudio.com/Download</a>

The theme I am using is Plainwhite, available from:

<a href="https://github.com/samarsault/plainwhite-jekyll">https://github.com/samarsault/plainwhite-jekyll</a>

Jekyll themes are tightly coupled to Jekyll the site generator so this is the entire required package.

The first steps are:

- Create Azure storage account and enable static website.

- Create Azure DevOps project
    - Import code into Repo
    - Add build.yml
    - Create Build pipeline
    - Create Release pipeline
    - Enable CI/CD so build and release pipelines are run whenever code changes are committed

Then look at domain name and SSL.

Then look at IaC to go from a blank slate to a fully deployed website in Azure.

Other notes:

- To reload tab without cache on OSX, use CMD/Shift/R


