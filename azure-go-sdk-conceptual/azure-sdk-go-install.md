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

### Azure service versions and profiles

The SDK is versioned semantically, but Azure services themselves are versioned on deployment dates. There are also regularly generated
_profiles_, which are dated snapshots of active services at a specific time. All services within a profile are _guaranteed_ to be compatible. 
There is no interoperability guarantee with service versions outside the profile.

For this reason, service versions and profiles are important when working with Azure. They are exposed through the Go SDK as part of the
import path for service modules. Be sure that you import compatible versions throughout any single project. Doing otherwise could introduce inconsistent behavior. The best way to guarantee compatibility is to use only `profile` modules.

Profiles are contained in the `profiles` module, and versioned as `YYYY-mm-dd`, `latest`, or `preview`. `latest` is always guaranteed to be the 
latest _stable_ profile, and `preview` is the release candidate for the next profile. Services are grouped underneath the profile version. For example, to import 
the DNS management module for the latest profile:

```go
import "github.com/Azure/azure-sdk-for-go/profiles/latest/dns/mgmt/dns"
```

If you have a need to import an individual service at a specific version, they are contained within the `services` module and have their version 
written as `YYYY-MM-dd`. Unlike profiles, versions are grouped under the service name. For example, to include the `2017-03-30` version of the `compute` module:

```go
import "github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2017-03-30/compute"
```

## Next steps

To begin using the Azure SDK for Go, try out a quickstart or look at some samples.

* [Deploy an Azure Virtual Machine from a template with the Go SDK](azure-sdk-go-qs-vm.md)

* [All samples for the Azure SDK for Go](https://github.com/azure-samples/azure-sdk-for-go-samples)
