---
title: Installing and use Azure Go SDK profiles on Azure Stack
description: How to install and use Azure Go SDK profiles on Azure Stack environment.
keywords: azure, azure stack, sdk, go, golang, azure sdk
author: seyadava
ms.author: seyadava
ms.date: 03/22/2018
ms.topic: article
ms.devlang: go
manager: knithinc
---

# What is a profile?
A profile is a combination of different resource types with different versions from different services. Using a profile will help you mix and match between various resource types. Profiles can provide:

* Stability for your application by locking to specific API versions; and/or
* Compatibility for your application with Azure Stack and regional Azure datacenters.

In Go SDK, profiles are available under the profiles/ path, with their version in the “YYYY-MM-DD” format. Right now, the latest Azure Stack profile version is “2017-03-09”. To import a given service from a profile, you need to import its corresponding module from the profile. For example, to import “Compute” service from “2017-03-09” profile:

```go
import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/compute/mgmt/compute"
```

# Install Azure SDK for Go
