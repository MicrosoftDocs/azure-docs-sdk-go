---
title: Tools for Go developers 
description: Tools for working with the Azure SDK for Go, and Azure services
keywords: azure, go, golang, azure, visual studio, visual studio code
author: sptramer
ms.author: sttramer
ms.date: 01/30/18
ms.topic: article
ms.devlang: go
manager: routlaw
---

# Tools for developers using the Azure SDK for Go

To be effective at writing Go code and have it work seamlessly with Azure services, here are some recommended tools.

## Azure CLI 2.0

The Azure 2.0 CLI provides a command-line interface to create and configure Azure resources in your subscriptions. The CLI can help you get started building common and shared Azure resources quickly, so that you can focus on more complex usage of services. The CLI has query and filtering features so you can pipe output directly to your favorite command-line tools. The CLI is available for installation on your local system, as a Docker image, or through [Azure Cloud Shell](https://docs.microsoft.com/en-us/azure/cloud-shell/overview).

> [!div class="nextstepaction"]
> [Install the Azure CLI 2.0](/cli/azure/install-azure-cli)

## Dependency management with go dep

There are many ways to manage your package dependencies and do vendoring with Go, since there is no official solution yet. The
recommended way to perform this management is with the `dep` dependency manager. 
The Azure SDK for Go uses dep for its vendoring, and is guaranteed to correctly get dependencies for any other project using dep.

> [!div class="nextstepaction"]
> [Get the dep dependency manager](https://github.com/tools/godep)

## Visual Studio Code

Visual Studio Code is a lightweight editor that has comprehensive support for the Go language through extensions. These extensions 
include support for features like autocomplete, `impl` templates, refactoring, and debugging. Visual Studio Code also offers many 
extensions for common developer tools such as source control, and even offers extensions for direct interactions with Azure services. If you use the Azure CLI, we also recommend installing the Azure CLI Tools extension.

* [Install Visual Studio Code](https://code.visualstudio.com/Download)
* [Install the Visual Studio Code Go extension](https://code.visualstudio.com/docs/languages/go)
* [Install the Azure CLI Tools extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)

## Go in Visual Studio

Microsoft only offers official Go support for Visual Studio Code.  If you require Go support for another Visual Studio version, 
you can find unofficial plugins in the [Visual Studio Marketplace](https://marketplace.visualstudio.com/vs).

