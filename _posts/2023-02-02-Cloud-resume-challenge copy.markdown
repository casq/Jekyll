---
layout: post
title:  "Cloud Resume Challenge"
date:   2023-04-30 15:40:00 +0000
categories: Jekyll CloudResumeChallenge
---

The goal of this project is to deploy a Jekyll website to Azure blob storage using Azure DevOps CI/CD, and be able to edit the site locally using VSCode.

This is part of the Cloud Resume Challenge for Azure:

<a href="https://cloudresumechallenge.dev/docs/the-challenge/azure">https://cloudresumechallenge.dev/docs/the-challenge/azure</a>

The theme I am using is Plainwhite, available from:

<a href="https://github.com/samarsault/plainwhite-jekyll">https://github.com/samarsault/plainwhite-jekyll</a>

Jekyll themes are tightly coupled to Jekyll the site generator so this is the entire required package.

The steps are:

- Set up storage account in Azure and enable static website
- Create an Azure DevOps organisation and project
    - Apply for parallelism grant from MS
    - Import Jekyll code into repository
    - Add build.yml
    - Create Build pipeline
    - Create Release pipeline
- Sync code locally