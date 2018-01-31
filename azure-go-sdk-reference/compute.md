
---
title: Azure Compute modules for Go
description: Reference documentation for the Azure Compute Go module
keywords: Azure, go, golang, SDK, API, compute, virtual machine, vm, scale set, 
author: sptramer
ms.author: sttramer
manager: routlaw
ms.date: 01/31/18
ms.topic: reference
ms.prod: azure
ms.technology: azure
ms.devlang: go
ms.service: cloud-services
---

# Azure Compute modules for Go

## Overview

The Azure Compute module provides an interface for working with virtual machines,
scale sets, and other computation-focused resources.


## Management API 

Create, delete, and manage virtual machine and virtual machine scale set resources in Azure.

```go
import "github.com/Azure/azure-sdk-for-go/profiles/latest/compute/mgmt/compute"
```

> [!div class="nextstepaction"]
> [Full management API reference on GoDoc](https://godoc.org/github.com/Azure/azure-sdk-for-go/profiles/latest/compute/mgmt/compute)

## Sample code 

* [Create a virtual machine with an existing NIC](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/compute/compute.go)
* [Create a virtual machine with a managed service identity](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/compute/vmmsi.go)

> [!div class="nextstepaction"]
> [View all Azure SDK for Go compute samples](https://github.com/Azure-Samples/azure-sdk-for-go-samples/compute)

