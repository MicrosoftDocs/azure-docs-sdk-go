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
To get the Authorizer property from Azure Active Directory using Go SDK, install the Go-AutoRest modules. These modules should have been already installed with the "Go SDK" installation; if not, install the [authentication package](https://github.com/Azure/go-autorest/tree/master/autorest/adal)
The Authorizer must be set as the authorizer for the resource client. There are different methods to get an Authorizer; for a complete list see [here](https://github.com/Azure/go-autorest/tree/master/autorest/adal#acquire-access-token).

This section presents a common way to get authorizer tokens on Azure Stack by using client credentials:

1. If a service principal with owner role on the subscription is available, skip this step. Otherwise create a service principal ([instructions](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals)) and assign it an "owner" role scoped to your subscription ([instructions](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-create-service-principals#assign-role-to-service-principal)). Save the service principal application ID and secret.
2. Import `adal` package from Go-AutoRest in your code. 

    ```go
    package main

    import "github.com/Azure/go-autorest/autorest/adal"
    ```
3. Create an oauthConfig by using NewOAuthConfig method from `adal` module. 

    ```go
    package main

    import "github.com/Azure/go-autorest/autorest/adal"

    func CreateToken() (adal.OAuthTokenProvider, error) {
        var token adal.OAuthTokenProvider
        oauthConfig, err := adal.NewOAuthConfig("<activeDirectoryEndpoint>", "<tenantID>")
    }
    ```

    Set `activeDirectoryEndpoint` to the value of `loginEndpoint` property that is retreived from the `resource manager endpoint` metadata on the previous section of this document.

    Set `tenantID` value to your Azure Stack tenant ID.

4. Finally, create a service principal token by using NewServicePrincipalToken method from `adal` module.

    ```go
    package main

    import "github.com/Azure/go-autorest/autorest/adal"

    func CreateToken() (adal.OAuthTokenProvider, error) {
        var token adal.OAuthTokenProvider
        oauthConfig, err := adal.NewOAuthConfig("<activeDirectoryEndpoint>", "<tenantID>")
        token, err = adal.NewServicePrincipalToken(
            *oauthConfig,
            "<clientID>",
            "<clientSecret>",
            "<tokenAudience>")
        return token, err
    }
    ```

    Set `tokenAudience` to one of the values in the `audience` list from the `resource manager endpoint` metadata retrieved on the previous section of this document.

    Set `clientID` to the service principal application ID saved when service principal was created on the previous section of this document.

    Set `clientSecret` to the service principal application Secret saved when service principal was created on the previous section of this document.

# Example
This section shows a sample of Go code to create virtual network on Azure Stack. For complete examples of Go SDK see [Azure Go SDK samples repository](https://github.com/Azure-Samples/azure-sdk-for-go-samples). Azure stack samples are available under hybrid/ path inside service folders of the repository.

Note: to run the code in this section, verify that your subscription has "Network" resource provider listed as "Registered." To verify it, look for the Subscription in the Azure Stack portal, and click on "Resource providers."

1. Import required packages in your code. You should use the latest available profile on Azure Stack to import the network module.

    ```go
    package main

    import (
        "context"
        "fmt"
        "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
        "github.com/Azure/go-autorest/autorest"
        "github.com/Azure/go-autorest/autorest/adal"
        "github.com/Azure/go-autorest/autorest/to"
    )
    ```

2. Define your environment variables. Note that to create a virtual network you need to have a resource group.

    ```go
    package main

    import (
        "context"
        "fmt"
        "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
        "github.com/Azure/go-autorest/autorest"
        "github.com/Azure/go-autorest/autorest/adal"
        "github.com/Azure/go-autorest/autorest/to"
    )

    var (
        activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
        tenantID = "yourAzureStackTenantID"
        clientID = "yourServicePrincipalApplicationID"
        clientSecret = "yourServicePrincipalSecret"
        activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
        subscriptionID = "yourSubscriptionID"
        baseURI = "yourResourceManagerURL"
        resourceGroupName = "existingResourceGroupName"
    )
    ```

3. Now that you have defined your environment variables, add a method to create authentication token by using adal package. See details about authentication in previous section.

    ```go
    //CreateToken creates a service principal token
    func CreateToken() (adal.OAuthTokenProvider, error) {
        var token adal.OAuthTokenProvider
        oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
        token, err = adal.NewServicePrincipalToken(
            *oauthConfig,
            clientID,
            clientSecret,
            activeDirectoryResourceID)
        return token, err
    }
    ```

4. Add the main method. The main method first gets a token by using the method that is defined in previous step. Then, it creates a client by using network module from profile. Finally, it creates a virtual network. 

    ```go
    package main

    import (
        "context"
        "fmt"
        "github.com/Azure/azure-sdk-for-go/profiles/2017-03-09/network/mgmt/network"
        "github.com/Azure/go-autorest/autorest"
        "github.com/Azure/go-autorest/autorest/adal"
        "github.com/Azure/go-autorest/autorest/to"
    )

    var (
        activeDirectoryEndpoint = "yourLoginEndpointFromResourceManagerUrlMetadata"
        tenantID = "yourAzureStackTenantID"
        clientID = "yourServicePrincipalApplicationID"
        clientSecret = "yourServicePrincipalSecret"
        activeDirectoryResourceID = "yourAudienceFromResourceManagerUrlMetadata"
        subscriptionID = "yourSubscriptionID"
        baseURI = "yourResourceManagerURL"
        resourceGroupName = "existingResourceGroupName"
    )

    //CreateToken creates a service principal token
    func CreateToken() (adal.OAuthTokenProvider, error) {
        var token adal.OAuthTokenProvider
        oauthConfig, err := adal.NewOAuthConfig(activeDirectoryEndpoint, tenantID)
        token, err = adal.NewServicePrincipalToken(
            *oauthConfig,
            clientID,
            clientSecret,
            activeDirectoryResourceID)
        return token, err
    }

    func main() {
        token, _ := CreateToken()
        vnetClient := network.NewVirtualNetworksClientWithBaseURI(baseURI, subscriptionID)
        vnetClient.Authorizer = autorest.NewBearerAuthorizer(token)
        future, _ := vnetClient.CreateOrUpdate(
            context.Background(),
            resourceGroupName,
            "sampleVnetName",
            network.VirtualNetwork{
                Location: to.StringPtr("local"),
                VirtualNetworkPropertiesFormat: &network.VirtualNetworkPropertiesFormat{
                    AddressSpace: &network.AddressSpace{
                        AddressPrefixes: &[]string{"10.0.0.0/8"},
                    },
                    Subnets: &[]network.Subnet{
                        {
                            Name: to.StringPtr("subnetName"),
                            SubnetPropertiesFormat: &network.SubnetPropertiesFormat{
                                AddressPrefix: to.StringPtr("10.0.0.0/16"),
                            },
                        },
                    },
                },
            })
        err := future.WaitForCompletion(context.Background(), vnetClient.Client)
        if err != nil {
            fmt.Printf(err.Error())
            return
        }
    }
    ```

