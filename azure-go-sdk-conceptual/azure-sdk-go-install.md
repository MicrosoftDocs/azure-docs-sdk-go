---
# Mandatory fields. See more on aka.ms/skyeye/meta.
title: Installing the Azure SDK for Go
description: How to install, vendor, and configure the Azure SDK for Go.
keywords: azure, sdk, go, golang, azure sdk
author: sptramer
ms.author: sttramer
ms.date: 12/23/2017
ms.topic: article
ms.devlang: golang
manager: routlaw
---

# Installing the Azure SDK for Go

Welcome to the Azure SDK for Go! This SDK allows you to manage Azure services from your Go applications and take advantage of client-side interactions with them. With the Go SDK you can manage [virtual machines](https://azure.microsoft.com/en-us/services/virtual-machines/), boot up [Azure Container Instances](https://azure.microsoft.com/en-us/services/container-instances/), access [Azure Storage](https://azure.microsoft.com/en-us/services/storage/), manage secrets in [Azure Key Vault](https://azure.microsoft.com/en-us/services/key-vault/), and much more.

[!INCLUDE [azure-sdk-go-get](../../../includes/azure-sdk-go-get)]

There is no additional configuration required for your environment. All of the information needed to connect to and use Azure services 
is performed through code.

## Including the Azure SDK for Go in your project

To access the SDK within your Go code and perform authentication you need to import modules for the accessed Azure services,
and the generated `autorest` modules they rely on. You can get a complete list of the available modules from GoDoc for 
[available services](https://godoc.org/github.com/Azure/azure-sdk-for-go) and 
[AutoRest packages](https://godoc.org/github.com/Azure/go-autorest). The packages that are required for any code which
will interact with Azure services are the following.

| Package | Description |
|---------|-------------|
| github.com/Azure/go-autorest/autorest/adal | Authentication mechanisms for accessing Azure services |
| github.com/Azure/go-autorest/autorest/to | Type-conversion helper functions for easily working with Azure SDK data structures |

### Azure service versions and profiles

While the SDK is versioned semantically, Azure services themselves are versioned on deployment dates. There are also regularly generated
_profiles_, which are dates when there is a snapshot of the active running service versions. All services within a profile are _guaranteed_ to be compatible. There is no interoperability guarantee with other service versions outside the profile.

For this reason, service versions and profiles are important when working with Azure. They are exposed through the Go SDK as part of the
import path for any service that you interact with. Care should be taken to make sure that the _same version profile_ is imported
throughout any single project -- doing otherwise could introduce inconsistent behavior, as returned values from one Azure service
may end up not being compatible as inputs to another service, if they are not from the same profile.

Individual services can be imported by version, written as `YYYY-MM-dd`. For example to include the `2017-03-30` version of the `compute modile`, you use the following import.

```golang
import "github.com/Azure/azure-sdk-for-go/services/compute/mgmt/2017-03-30/compute"
```

Profiles are written out as `YYYY-mm-dd`, `latest`, or `preview`. `latest` is always guaranteed to be the latest _stable_ profile, and `preview` is the release candidate for the next profile. Profiles are all contained within the `profiles` module, and have services grouped underneath their version. For example, to import the DNS management module for the latest profile,
you use the following import.

```golang
import "github.com/Azure/azure-sdk-for-go/profiles/latest/dns/mgmt/dns"
```

> [!IMPORTANT]
> Any time you import a module with `github.com/Azure` as part of its path, `Azure` _must_ be capitalized. This is because it is captalized in dependent libraries.

## Next steps

To begin using the Azure SDK for Go try a quickstart, look at authentication, or learn effective
ways to work with the `Future` type.

* [Quickstart: Deploy an Azure Virtual Machine from a template with the Go SDK](azure-sdk-go-qs-vm.md)
* [How-to: Working with Future in Azure SDK for Go](azure-sdk-go-future.md)
* [How-to: Authenticating your Go application with Azure Services](azure-sdk-go-auth.md)
