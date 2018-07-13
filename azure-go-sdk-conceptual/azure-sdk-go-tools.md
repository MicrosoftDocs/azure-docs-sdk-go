---
title: Tools for Go developers 
description: Tools for working with the Azure SDK for Go, and Azure services
author: sptramer
ms.author: sttramer
manager: carmonm
ms.date: 07/13/2018
ms.topic: conceptual
ms.prod: azure
ms.technology: azure-sdk-go
ms.devlang: go
---

# Tools for developers using the Azure SDK for Go

To be effective at writing Go code and have it work seamlessly with Azure services, here are some recommended tools.

## Azure CLI

The Azure CLI provides a command-line interface to create and configure Azure resources in your subscriptions. The CLI can help you get started building common and shared Azure resources quickly, so that you can focus on more complex usage of services. The CLI has query and filtering features so you can pipe output directly to your favorite command-line tools. The CLI is available for installation on your local system, as a Docker image, or through [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview).

> [!div class="nextstepaction"]
> [Install the Azure CLI](/cli/azure/install-azure-cli)

## Visual Studio Code

Visual Studio Code is a lightweight editor that has comprehensive support for the Go language through extensions. These extensions 
include support for features like autocomplete, `impl` templates, refactoring, and debugging. Visual Studio Code also offers many 
extensions for common developer tools such as source control, and even offers extensions for direct interactions with Azure services. 
Microsoft maintains an official meta-extension including these Azure extensions, including an interactive interface for the Azure CLI.

* [Install Visual Studio Code](https://code.visualstudio.com/Download)
* [Get the Visual Studio Code Go extension](https://code.visualstudio.com/docs/languages/go)
* [Get the Azure Tools extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-azureextensionpack)

## CI/CD with Azure DevOps Project

With the Azure DevOps Project pipeline, you can set up a continuous build and deploy for your Go applications. All you need is an available git
repo, and you can set up to deploy and test directly on your Azure resources. The configuration pipeline is easy to create and manage,
and since it's provisioned directly on Azure, you can control it in the same way that you handle your other Azure resources.

> [!div class="nextstepaction"]
> [Learn how to create a CI/CD pipeline with Azure DevOps Project](/devops-project/azure-devops-project-go)

## Dependency management with dep

There are many ways to manage your package dependencies and do vendoring with Go, since there is no official solution yet. The
recommended way to perform this management is with the `dep` dependency manager. 
The Azure SDK for Go uses dep for its vendoring, and is guaranteed to correctly get dependencies for any other project using dep.

> [!div class="nextstepaction"]
> [Get the dep dependency manager](https://github.com/golang/dep)
