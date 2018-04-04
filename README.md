# CI/CD Demo - Wercker Container Pipeline

This repository contains the code for the WorkBetter application demo. The demo will Dockerize this application using Wercker, configure a CI/CD pipeline, and deploy the application to a Kubernetes cluster running on Oracle Cloud Infrastructure.

On every pipeline execution, the code goes through the following steps:

1. Code is pulled from Github, built, tested and deployed to Kubernetes
2. The Docker image is pushed to the internal Wercker registry
3. The application is then tested
3. If passed, the image will be pulled from the registy and deployed to the Kubernetes cluster


WorkBetter is a complete sample app showcasing the capabilities of JET for building web applications. WorkBetter has been designed for use as a web application in browsers from a mobile phone up to desktop. WorkBetter demonstrates web UI patterns and best practices, including Routing, ojModule, and data resource interactions.
