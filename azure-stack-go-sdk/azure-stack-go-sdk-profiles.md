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
3. Install Go SDK and its dependencies by running the following bash command:

    ```bash
    go get -u -d github.com/Azure/azure-sdk-for-go/...
    ```
    * Full details for Azure Go SDK Installation are [here](https://docs.microsoft.com/en-us/go/azure/azure-sdk-go-install#get-the-azure-sdk-for-go)
    * Azure Go SDK is publicly available at [Go SDK GitHub](https://github.com/Azure/azure-sdk-for-go)
    * Go SDK uses Go-AutoRest modules to send requests to Azure service endpoints. To use Azure services in code, in addition to Azure Go SDK, modules must be imported from [Go-AutoRest](https://github.com/Azure/go-autorest). More details about the most common Go-Autorest packages can be found [here](https://docs.microsoft.com/en-us/go/azure/azure-sdk-go-install#including-the-azure-sdk-for-go-in-your-project). For a complete list of the available modules in Go-AutoRest go [here](https://godoc.org/github.com/Azure/go-autorest).

# How to use GO SDK profiles on Azure Stack
To run a sample of Go code on Azure Stack:

1. Install Azure SDK for Go and its dependencies (see previous section)
2. Get metadata information from resource manager endpoint. Resource manager endpoint returns a JSON file with required information to run Go samples.
    * Note:
        * Resource manager endpoint in one node environment is: https://management.local.azurestack.external/
        * Resource manager endpoint in multi node environment is: https://management.`LOCATION`.ext-`MACHINE_NAME`.masd.stbtest.microsoft.com/'

    To retrieve the metadata go to: ResourceManagerEndpoint/metadata/endpoints?api-version=1.0

    Sample JSON file from a one node environment (https://management.local.azurestack.external/metadata/endpoints?api-version=1.0): 

    ```json
    {
        "galleryEndpoint": "https://portal.local.azurestack.external:30015/",
        "graphEndpoint": "https://graph.windows.net/",
        "portalEndpoint": "https://portal.local.azurestack.external/",
        "authentication": {
            "loginEndpoint": "https://login.windows.net/",
            "audiences": [
                "https://management.azurestackiXX.onmicrosoft.com/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
            ]
        }
    }
    ```
3. If not available, create a subscription and save the subscription ID to be used later. Instructions to create a subscription are [here](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals)
4. Create a service principal with "Subscription" scope and "Owner" role. Save the service principal's ID and secret. Instructions to create a service principal for Azure Stack are [here](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals)
5. Now your Azure Stack environment is setup. In your code, import a service module from Go SDK profile. The current version of Azure Stack profile is "2017-03-09". For example, to import network module from "2017-03-09" profile:

    ```go
    package main

    import "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
    ```

6. In your function, create and authenticate a client with a New*Client function call. For example to create a virtual network client:

    ```go
    package main

    import (
    "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
    "github.com/Azure/go-autorest/autorest"  
    ) 

    func main() {
        vnetClient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "<subscriptionID>")
        vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
    }
    ```

    Set `baseURI` to the `ResourceManagerEndpoint` value used in step 2.
    Set `subscriptionID` to the subscription ID value from step 3.
    To create token see Authentication section below

7. Invoke API methods by using the client that you created in the previous step. For example, to create a virtual network by using our client from previous step:

    ```go
    package main

    import (
    "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
    "github.com/Azure/go-autorest/autorest"  
    ) 

    func main() {
        vnetClient := network.NewVirtualNetworksClientWithBaseURI("<baseURI>", "<subscriptionID>")
        vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)

        vnetClient.CreateOrUpdate(...)
    }
    ```
    For a complete example of creating a virtual network on Azure Stack by using Go SDK profile, see Example section below.

# Authentication