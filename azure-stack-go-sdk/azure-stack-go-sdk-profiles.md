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

# Install Azure SDK for Go on Azure Stack
1. Follow the official instructions to install [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
2. Follow the official instructions to install [Go](https://golang.org/dl/). Note that to use profiles you need to install Go version 1.9 or newer.
3. 	3. Install Go SDK and its dependencies by running the following bash command:

    ```bash
    go get -u -d github.com/Azure/azure-sdk-for-go/...
    ```
    * Full details for Azure Go SDK Installation are [here](https://docs.microsoft.com/en-us/go/azure/azure-sdk-go-install#get-the-azure-sdk-for-go)
    * Azure Go SDK is publicly available at [Go SDK GitHub](https://github.com/Azure/azure-sdk-for-go)
    * Go SDK uses Go-AutoRest modules to send requests to Azure service endpoints. To use Azure services in code, in addition to Azure Go SDK, modules must be imported from [Go-AutoRest](https://github.com/Azure/go-autorest). More details about the most common Go-Autorest packages can be found [here](https://docs.microsoft.com/en-us/go/azure/azure-sdk-go-install#including-the-azure-sdk-for-go-in-your-project). For a complete list of the available modules in Go-AutoRest go [here](https://godoc.org/github.com/Azure/go-autorest).

# How to use GO SDK profiles on Azure Stack
To run a sample of Go code on Azure Stack:

1. Install Azure SDK for Go and its dependencies (see previous section)
2. Get metadata information from resource manager endpoint; which will return a JSON file with required information to run Go samples.
    Note:   ResourceManagerUrl in one node environment is: (https://management.local.azurestack.external/)
            ResourceManagerUrl in multi node environment is: (https://management.<location>.ext-<machine-name>.masd.stbtest.microsoft.com/)

    To retrieve the metadata go to: <ResourceManagerUrl>/metadata/endpoints?api-version=1.0

    Sample JSON file: 
    