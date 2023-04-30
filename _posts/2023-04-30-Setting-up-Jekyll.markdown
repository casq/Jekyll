---
layout: post
title:  "Cloud Resume Challenge"
date:   2023-04-30 00:15:40 +0000
categories: Jekyll CloudResumeChallenge
---

The goal of this project is to deploy a Jekyll website to Azure blob storage using Azure DevOps CI/CD, and be able to edit the site locally using VSCode.

This is part of the Cloud Resume Challenge for Azure:

<a href="https://cloudresumechallenge.dev/docs/the-challenge/azure">https://cloudresumechallenge.dev/docs/the-challenge/azure</a>

The theme I am using is Plainwhite, available from:

<a href="https://github.com/samarsault/plainwhite-jekyll">https://github.com/samarsault/plainwhite-jekyll</a>

Jekyll themes are tightly coupled to Jekyll the site generator so this is the entire required package.

The steps are:

- Fork Plainwhite in GitHub

- Create Azure storage account and enable static website.

- Create Azure DevOps project
    - Import code into Repo
    - Add build.yml
    - Create Build pipeline
    - Create Release pipeline
    - Enable CI/CD so build and release pipelines are run whenever code changes are committed