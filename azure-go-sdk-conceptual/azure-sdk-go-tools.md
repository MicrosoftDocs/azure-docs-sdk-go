---
title: Tools for Go developers 
description: Tools for working with the Azure SDK for Go, and Azure services
keywords: ???
author: sptramer
ms.author: sttramer
ms.date: 10/03/2017
ms.topic: article
ms.devlang: golang
manager: routlaw
---

# Tools for developers using the Azure SDK for Go

To be effective at writing Go code and have it work seamlessly with Azure services, here are some recommended tools.

## Azure CLI 2.0

The Azure 2.0 CLI provides a command line interface to create and configure Azure resources in your subscriptions. The CLI can help you get started building common and shared Azure resources quickly, so that you can focus on more complex usage of services. The CLI has query and filtering features so you can pipe output directly to your favorite command line tools. You can install the CLI on macOS, Windows, and Linux.

[Install the Azure CLI 2.0](/cli/azure/install-azure-cli)

## Go dependency management with go dep

There are many ways to manage your package dependencies and do vendoring with Go, since there is no official solution yet. The
recommended ways to perform this management is with the official [dep](https://github.com/tools/godep) dependency manager. 
The Azure SDK for Go uses dep for its vendoring, and is guaranteed to correctly get dependencies for any other project using dep.

## Visual Studio Code

Visual Studio Code is a lightweight editor that has comprehensive support for the Go language through extensions. These extensions 
include support for features like autocomplete, `impl` templates, refactoring, and debugging. Visual Studio Code also offers many 
extensions for common developer tools such as source control, and even offers extensions for direct interactions with Azure services.

* [Install Visual Studio Code](https://code.visualstudio.com/Download)
* [Install the Visual Studio Code Go extension](https://code.visualstudio.com/docs/languages/go)

If you will be using the Azure CLI for working with Azure resources outside of your code, we also recommend installing the
[Azure CLI Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli) plugin.

## Go in Visual Studio

Microsoft only offers official Go support for Visual Studio Code.  If you require Go support for another Visual Studio version, 
you can find unofficial plugins in the [Visual Studio Marketplace](https://marketplace.visualstudio.com/vs).

