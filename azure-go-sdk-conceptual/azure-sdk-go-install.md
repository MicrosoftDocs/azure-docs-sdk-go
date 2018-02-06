---
title: Installing the Azure SDK for Go
description: How to install, vendor, and configure the Azure SDK for Go.
keywords: azure, sdk, go, golang, azure sdk
author: sptramer
ms.author: sttramer
ms.date: 01/30/18
ms.topic: article
ms.devlang: go
manager: routlaw
---

# Installing the Azure SDK for Go

Welcome to the Azure SDK for Go! This SDK allows you to manage and interact with Azure services from your Go applications.

[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

Working with Azure Storage Blobs requires a separate SDK.

```bash
go get -u -d github.com/Azure/azure-storage-blob-go/...
```

## Vendoring the Azure SDK for Go

The Azure SDK for Go may be vendored through [dep](https://github.com/golang/dep). For stability reasons, vendoring is recommended. In order
to use `dep` support, add `gitub.com/Azure/azure-sdk-for-go` to a `[[constraint]]` section of your `Gopkg.toml`. For example, to vendor on version `12.0.0-beta`:

```
[[constraint]]
name = "github.com/Azure/azure-sdk-for-go"
version = "12.0.0-beta"
```

## Including the Azure SDK for Go in your project

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
and are from either a _service version_ or a _profile_. For stability purposes, you should choose a single profile version and import all
services from that profile. Profiles are located under the `profiles` module, and named after their version. Stable profiles are versioned
on the date they were created, in `YYYY-MM-DD` format. Services are grouped under their profile version.

For example, to import the Azure Resoruces management module from the `2017-03-09` profile:

```go
import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/resources/mgmt/resources"
```

> [!WARNING]
> It is not recommended to use the `preview` or `latest` profiles, since these are rolling versions and service behavior may change at any time.

If you have a need for a specific version of a service, these are located under the `services` module, followed by the service name, and then
the service version in `YYYY-MM-DD` format. For example, to include the `2017-03-30` version of the Compute service:

```go
import "github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2017-03-30/compute"
```

## Next steps

To begin using the Azure SDK for Go, try out a quickstart or look at some samples.

* [Deploy an Azure Virtual Machine from a template with the Go SDK](azure-sdk-go-qs-vm.md)

* [All samples for the Azure SDK for Go](https://github.com/azure-samples/azure-sdk-for-go-samples)
