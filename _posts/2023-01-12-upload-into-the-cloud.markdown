---
layout: post
title:  "Upload into the cloud"
date:   2023-01-12 12:53:36 +0530
categories: Jekyll GettingStarted
---
Uploading into Azure Storage

- Need to have Azure Storage extension in VSCode
- Use command palette if you need to sign out of an existing subscription - Cmd-Shift-P
- Create storage account in Azure and enable static website - Data Management -> Static Website
- This creates a $web blob to which your Azure DevOps pipeline will upload the contents of your local Jekyll _site folder