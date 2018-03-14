---
title: Install the Azure SDK for Go
description: How to install, vendor, and configure the Azure SDK for Go.
author: sptramer
ms.author: sttramer
ms.date: 03/14/2018
ms.topic: article
ms.devlang: go
manager: carmonm
---

# Install the Azure SDK for Go

Welcome to the Azure SDK for Go! This SDK allows you to manage and interact with Azure services from your Go applications.

## Get the Azure SDK for Go

[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

Working with Azure Storage Blobs requires a separate SDK.

```bash
go get -u -d github.com/Azure/azure-storage-blob-go/...
```

## Vendor the Azure SDK for Go

The Azure SDK for Go may be vendored through [dep](https://github.com/golang/dep). For stability reasons, vendoring is recommended. In order
to use `dep` support, add `github.com/Azure/azure-sdk-for-go` to a `[[constraint]]` section of your `Gopkg.toml`. For example, to vendor on version `14.0.0`, add the following entry:

```
[[constraint]]
name = "github.com/Azure/azure-sdk-for-go"
version = "14.0.0"
```

## Include the Azure SDK for Go in your project

To use Azure services from your Go code, import any services you interact with and the required `autorest` modules.
 You get a complete list of the available modules from GoDoc for 
[available services](https://godoc.org/github.com/Azure/azure-sdk-for-go) and 
[AutoRest packages](https://godoc.org/github.com/Azure/go-autorest). The most common packages you need from `go-autorest`
are:

| Package | Description |
|---------|-------------|
| [github.com/Azure/go-autorest/autorest][autorest] | Objects for handling service client authentication |
| [github.com/Azure/go-autorest/autorest/azure][autorest/azure] | Constants for interactions with Azure services |
| [github.com/Azure/go-autorest/autorest/adal][autorest/adal] | Authentication mechanisms for accessing Azure services |
| [github.com/Azure/go-autorest/autorest/to][autorest/to] | Type assertion helpers for working with Azure SDK data structures |

[autorest]: https://godoc.org/github.com/Azure/go-autorest/autorest
[autorest/azure]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure
[autorest/adal]: https://godoc.org/github.com/Azure/go-autorest/autorest/adal
[autorest/to]: https://godoc.org/github.com/Azure/go-autorest/autorest/to

Modules for Azure services are versioned independently from the SDK APIs for them. These versions are part of the module import path,
and are from either a _service version_ or a _profile_. Currently, it is recommended that you use a specific service version for
both development and release. Services are located under the `services` module. The full path for the import is the name of the service, followed by
the version in `YYYY-MM-DD` format, followed by the service name again. For example, to include the `2017-03-30` version of the Compute service:

```go
import "github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2017-03-30/compute"
```

Right now it is recommended that you use the latest version of a service, unless you have a reason to do otherwise.

If you need a collective snapshot of services, you can also select a single profile version. Right now, the only locked profile is version 
`2017-03-30`, which may not have the latest features of services. Profiles are located under the `profiles` module, with their version in the `YYYY-MM-DD` format. 
Services are grouped under their profile version. For example, to import the Azure Resources management module from the `2017-03-09` profile:

```go
import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/resources/mgmt/resources"
```

> [!WARNING]
> There are also `preview` and `latest` profiles available. Using them is not recommended. These profiles are rolling versions and service behavior may change at any time.

## Next steps

To begin using the Azure SDK for Go, try out a quickstart.

* [Deploy a virtual machine from a template](azure-sdk-go-qs-vm.md)
* [Transfer objects to Azure Blob Storage with the Azure Blob SDK for Go](/azure/storage/blobs/storage-quickstart-blobs-go?toc=%2fgo%2fazure%2ftoc.json)
* [Connect to Azure Database for PostgreSQL](/azure/postgresql/connect-go?toc=%2fgo%2fazure%2ftoc.json)

If you want to get started with other services in the Go SDK immediately,
take a look at some of the available sample code.

* [Authenticate with Azure services](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/iam)
* [Deploy new virtual machines with SSH authentication](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/compute)
* [Deploy a container image to Azure Container Instances](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/containerinstance)
* [Create a cluster in Azure Kubernetes Service](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/containerservice)
* [Work with Azure Storage services](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/storage)
* [All samples for the Azure SDK for Go](https://github.com/azure-samples/azure-sdk-for-go-samples)
