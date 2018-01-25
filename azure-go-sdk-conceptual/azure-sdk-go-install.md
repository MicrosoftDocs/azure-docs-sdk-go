---
title: Installing the Azure SDK for Go
description: How to install, vendor, and configure the Azure SDK for Go.
keywords: azure, sdk, go, golang, azure sdk
author: sptramer
ms.author: sttramer
ms.date: 01/25/18
ms.topic: article
ms.devlang: golang
manager: routlaw
---

# Installing the Azure SDK for Go

Welcome to the Azure SDK for Go! This SDK allows you to manage Azure services from your Go applications and take advantage of client-side interactions with them. With the Go SDK you can manage [virtual machines](https://azure.microsoft.com/en-us/services/virtual-machines/), boot up [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/), access [Azure Storage](https://azure.microsoft.com/en-us/services/storage/), manage secrets in [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/), and much more.

[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

If you plan on working with Azure Storage Blobs, you will need to get the separate SDK for working with those services.

```bash
go get -u -d github.com/Auzre/azure-storage-blob-go/...
```

If you prefer, the Azure SDK for Go can also be vendored through [dep](https://github.com/golang/dep). Until the official GA release,
there may be breaking changes in the SDK, so it is strongly recommended that you select a specific version and use vendoring. In order
to use `go dep` support, add `gitub.com/Azure/azure-sdk-for-go` to a `[[constraint]]` section of your `Gopkg.toml`. For example, to vendor on version `12.0.0-beta`, add the following. 

```
[[constraint]]
name = "github.com/Azure/azure-sdk-for-go"
version = "12.0.0-beta"
```

There is no additional configuration required for your environment. 

## Including the Azure SDK for Go in your project

To access the SDK within your Go code and perform authentication you need to import modules for the accessed Azure services,
and the generated `autorest` modules they rely on. You can get a complete list of the available modules from GoDoc for 
[available services](https://godoc.org/github.com/Azure/azure-sdk-for-go) and 
[AutoRest packages](https://godoc.org/github.com/Azure/go-autorest). The packages that are required for any code which
will interact with Azure services are:

| Package | Description |
|---------|-------------|
| github.com/Azure/go-autorest/autorest | The base Autorest package, containing implementations required by the SDK |
| github.com/Azure/go-autorest/autorest/azure | Backing interactions with the Azure REST API |
| github.com/Azure/go-autorest/autorest/adal | Authentication mechanisms for accessing Azure services |
| github.com/Azure/go-autorest/autorest/to | Type-conversion helper functions for easily working with Azure SDK data structures |

### Azure service versions and profiles

While the SDK is versioned semantically, Azure services themselves are versioned on deployment dates. There are also regularly generated
_profiles_, which are dates when there is a snapshot of the active running service versions. All services within a profile are _guaranteed_ to be compatible. 
There is no interoperability guarantee with other service versions outside the profile.

For this reason, service versions and profiles are important when working with Azure. They are exposed through the Go SDK as part of the
import path for service modules. Care should be taken to make sure that compatible service versions are imported
throughout any single project -- doing otherwise could introduce inconsistent behavior, as returned values from one Azure service
may end up not being compatible as inputs to another service. The easiest way to guarantee compatibility is to use only `profile` modules.

Profiles are contained in the `profiles` module, and versioned as `YYYY-mm-dd`, `latest`, or `preview`. `latest` is always guaranteed to be the 
latest _stable_ profile, and `preview` is the release candidate for the next profile. Services are grouped underneath the profile version. For example, to import 
the DNS management module for the latest profile, use the following import.

```golang
import "github.com/Azure/azure-sdk-for-go/profiles/latest/dns/mgmt/dns"
```

If you have a need to import an individual service at a specific version, they are contained within the `services` module and have their version 
written as `YYYY-MM-dd`. Unlike profiles, versions are grouped under the service name. For example, to include the `2017-03-30` version of the `compute` module, use the following import.

```golang
import "github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2017-03-30/compute"
```

## Next steps

To begin using the Azure SDK for Go try a quickstart, look at authentication, or learn effective
ways to work with the `Future` type.

* [Quickstart: Deploy an Azure Virtual Machine from a template with the Go SDK](azure-sdk-go-qs-vm.md)
* [How-to: Working with Future in Azure SDK for Go](azure-sdk-go-future.md)
* [How-to: Authenticating your Go application with Azure Services](azure-sdk-go-auth.md)
