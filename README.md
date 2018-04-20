# CI/CD Demo - Wercker Container Pipeline

## Demo Introduction

This repository contains the code for the WorkBetter application demo. This demo will walk you through the process of moving an existing application into a containerised CI/CD pipeline and deploying it to a Kubernetes cluster running on Oracle Cloud Indfrastructure.

You will take on the role of a developer, responsible for configuring the automated build and deploy process that involve details about the application itself. To containerise and automate the building and deploying of this application you will make use of Wercker Pipelines, Oracle Container Registry, and Oracle Cloud Infrastrastructure. The Kubernetes infrastructure will be configured and provisioned through Terraform Installer.

On every pipeline execution, the code goes through the following steps:

1. Code is pulled from Github, built, tested and deployed to Kubernetes
2. The Docker image is pushed to the internal Wercker registry
3. The application is then tested
3. If passed, the image will be pulled from the registy and deployed to the Kubernetes cluster


## Time to Complete

Approximately 20 minutes


## What You Need?

- GitHub account
- Wercker account
- Oracle Cloud Infrastructure
- A Kubernetes cluster running on OCI
- An installation of Brackets with Bracket Git by Martin Zagora extension installed
- Some familiarity with GitHub
- Some familiarity with Wercker
- Some familiarity with Brackets



### **Step 1**: Obtain an Oracle Cloud Account



