---
title: Azure SDK for Go samples for compute and networking 
description: Selected samples for working with compute resources like VMs and virtual networks from the Azure SDK for Go.
author: sptramer
ms.author: sttramer
ms.date: 03/21/2018
ms.topic: article
ms.devlang: go
manager: carmonm
---

# Azure SDK for Go samples for compute and networking

The following table links to selected samples of Go source code that you can use to manage VMs, virtual networks, and subnets in Azure. To get all of the sample code and tests for compute and networking, use the `go get` command:

```bash
go get -d -u github.com/azure-samples/azure-sdk-for-go-samples/compute
go get -d -u github.com/azure-samples/azure-sdk-for-go-samples/network
```

All samples for the Azure SDK for Go are available on [GitHub](https://github.com/Azure-Samples/azure-sdk-for-go-samples).

| Name | Description |
|------|-------------|
| [network/network](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/network/network.go) | Create, update, delete, and query network resources including virtual networks, subnets, and network security groups. |
| [compute/loadbalancer](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/compute/loadbalancer.go) | Create and query availability groups, and create VMs with a load balancer. |
| [compute/compute](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/compute/compute.go) | Create, delete, update, and power-manage VMs. Work with data disks and managing the OS disk of the VM. |