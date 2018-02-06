---
title: Deploy an Azure virtual machine from Go 
description: Deploy a virutal machine using the Azure SDK for Go.
keywords: azure, virtual machine, vm, go, golang, azure sdk
author: sptramer
ms.author: sttramer
ms.date: 02/08/2018
ms.topic: quickstart
ms.devlang: go
manager: routlaw
---

# Quickstart: Deploy an Azure virtual machine from a template with the Azure SDK for Go

This quickstart focuses on deploying resources from a template with the Azure SDK for Go. Templates are snapshots of all of the resources contained within an [Azure resource group](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview). Along the way, you'll become familiar with the functionality and conventions of the SDK while performing a useful task.

At the end of this quickstart, you have a running VM that you log into with a username and password.

[!INCLUDE [quickstarts-free-trial-note](includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](includes/cloud-shell-try-it.md)]

If you use a local install of the Azure CLI, this quickstart requires CLI version 2.0.24 or later. Run `az --version` to make sure your CLI install meets this requirement. If you need to install or upgrade, see [Install the Azure CLI 2.0](/cli/azure/install-azure-cli).

## Install the Azure SDK for Go 

[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

## Create a service principal

To log in non-interactively with an application, you need a service principal. Service principals are part of Role-Based Authentication (RBAC), which creates a unique user identity. To create a new service principal with the CLI, run the following command:

```azurecli-interactive
az ad sp create-for-rbac --name az-go-vm-quickstart
```

__Make sure__ to record the `appId`, `password`, and `tenant` values in the output. These values are used by the application to authenticate with Azure.

For more information on creating and managing Service Principals with the Azure CLI 2.0, see [Create an Azure service principal with Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli).

## Get the code

Get the quickstart code and all of its dependencies with `go get`.

```bash
go get -u -d github.com/azure-samples/azure-sdk-for-go-samples/quickstart/deploy-vm/...
```

This code compiles, but doesn't run correctly until you provide it information about your Azure account and the created service principal. In `main.go` there is a variable, `config`, which contains an `authInfo` struct. This struct needs to have its field values replaced in order to authenticate correctly.

```go
var (
	config = authInfo{
		TenantID:               "",
		SubscriptionID:         "",
		ServicePrincipalID:     "",
		ServicePrincipalSecret: "",
	}
```

* `SubscriptionID`: Your subscription ID, which can be obtained from the CLI command

  ```azurecli-interactive
  az account show --query id -o tsv
  ```

* `TenantID`: Your tenant ID, the `tenant` value recorded when creating the service principal
* `ServicePrincipalID`: The `appId` value recorded when creating the service principal
* `ServicePrincipalSecret`: The `password` value recorded when creating the service principal

You also need to edit a value in the `vm-quickstart-params.json` file.

```json
    "vm_password": {
        "value": "_"
    }
```

* `vm_password`: The password for the VM user account. It must be 6-72 characters in length and contain 3 of the following characters:
  * A lowercase letter
  * An uppercase letter
  * A number
  * A symbol

## Running the code

Run the quickstart with the `go run` command.

```bash
cd $GOPATH/src/github.com/azure-samples/azure-sdk-for-go-samples/quickstart/deploy-vm
go run main.go
```

If there is a failure in the deployment, you get a message indicating that there was an issue, but without any specific details. Using the Azure CLI, get the details of the deployment failure with the following command:

```azurecli-interactive
az group deployment show -g GoVMQuickstart -n VMDeployQuickstart
```

If the deployment is successful, you see a message giving the username, IP address, and password for logging into the newly created virtual machine. SSH into this machine to confirm that it is up and running.

## Cleaning up

Clean up the resources created during this quickstart by deleting the resource group with the CLI.

```azurecli-interactive
az group delete -n GoVMQuickstart
```

## Code in depth

What the quickstart code does is broken down into a block of variables and several small functions, each of which are discussed here.

### Variable assignments and structs

Since quickstart is self-contained, it uses global variables rather than command-line options or environment variables.

```go
type authInfo struct {
        TenantID               string
        SubscriptionID         string
        ServicePrincipalID     string
        ServicePrincipalSecret string
}
```

The `authInfo` struct is declared to encapsulate all of the information needed for authorization with Azure services.

```go
const (
    resourceGroupName     = "GoVMQuickstart"
    resourceGroupLocation = "eastus"

    deploymentName = "VMDeployQuickstart"
    templateFile   = "vm-quickstart-template.json"
    parametersFile = "vm-quickstart-params.json"
)

var (
    config = authInfo{
        TenantID:               "",
        SubscriptionID:         "",
        ServicePrincipalID:     "",
        ServicePrincipalSecret: "",
    }

    ctx = context.Background()

    token *adal.ServicePrincipalToken
)
```

Values are declared which give the names of created resources. The location is also specified here, which you can change to see how deployments behave in other datacenters. Not every datacenter has all of the required resources available.

The `templateFile` and `parametersFile` constants point to the files needed for deployment. The service principal token is covered later, and the `ctx` variable is a [Go context](https://blog.golang.org/context) for the network operations.

### init() and authorization

The `init()` method for the code sets up authorization. Since authorization is a precondition for everything in the quickstart, it makes sense to have it as part of initialization. 

```go
func init() {
        // get OAuth token using Service Principal credentials
        oauthConfig, err := adal.NewOAuthConfig(azure.PublicCloud.ActiveDirectoryEndpoint, config.TenantID)
        if err != nil {
                log.Fatalf("%s: %v\n", "failed to get OAuth config", err)
        }
        token, err = adal.NewServicePrincipalToken(
                *oauthConfig,
                config.ServicePrincipalID,
                config.ServicePrincipalSecret,
                azure.PublicCloud.ResourceManagerEndpoint)
        if err != nil {
                log.Fatalf("faled to get token: %v\n", err)
        }
}
```

This code completes two steps for authorization:

* OAuth configuration information for the `TenantID` is retrieved by interfacing with Azure Active Directory. The [`azure.PublicCloud`](https://godoc.org/github.com/Azure/go-autorest/autorest/azure#PublicCloud) object contains endpoints used in the standard Azure configuration.
* The [`adal.NewServicePrincipalToken()`](https://godoc.org/github.com/Azure/go-autorest/autorest/adal#NewServicePrincipalToken) function is called. This function takes the OAuth information along with the service principal login, as well as which style of Azure management is being used. Unless you have specific requirements and know what you're doing, this value should always be `.ResourceManagerEndpoint`.

### Flow of operations in main()

The `main()` function is simple, only indicating the flow of operations and performing error-checking.

```go
func main() {
        group, err := createGroup()
        if err != nil {
                log.Fatalf("failed to create group: %v", err)
        }
        log.Printf("created group: %v\n", *group.Name)

        log.Printf("starting deployment\n")
        result, err := createDeployment()
        if err != nil {
                log.Fatalf("Failed to deploy correctly: %v", err)
        }
        log.Printf("Completed deployment: %v", *result.Name)
        getLogin()
}
```

The steps that the code runs through are, in order:

* Create the resource group to deploy to (`createGroup()`)
* Create the deployment within this group (`createDeployment()`)
* Obtain and display login information for the deployed VM (`getLogin()`)

### Creating the resource group

The `createGroup()` function creates the resource group. Looking at the call flow and arguments demonstrates the way that service interactions are structured in the SDK.

```go
func createGroup() (resources.Group, error) {
        groupsClient := resources.NewGroupsClient(config.SubscriptionID)
        groupsClient.Authorizer = autorest.NewBearerAuthorizer(token)

        return groupsClient.CreateOrUpdate(
                ctx,
                resourceGroupName,
                resources.Group{
                        Location: to.StringPtr(resourceGroupLocation)})
}
```

The general flow of interacting with an Azure service is:

* Create the client using the `service.NewXClient()` method, where `X` is the resource type of the `service` that you want to interact with. This function always takes a subscription ID.
* Set the authorization method for the client, allowing it to interact with the remote API.
* Make the method call on the client corresponding to the remote API. Service client methods usually take the name of the resource and a metadata object.

The [`to.StringPtr()`](https://godoc.org/github.com/Azure/go-autorest/autorest/to#StringPtr) function is used to perform a type conversion here. The parameters structs for methods of the SDK almost exclusively take pointers, so these methods are 
provided to make the type conversions easy. See the documentation for the [autorest/to](https://godoc.org/github.com/Azure/go-autorest/autorest/to) module for the complete list 
and behavior of convenience converters.

The `groupsClient.CreateOrUpdate()` operation returns a pointer to a data struct representing the resource group. A direct return value of this kind indicates a short-running operation that is meant to be synchronous. In the next section, you'll see an example of a long-running operation and how to interact with them.

### Performing the deployment

Once the group to contain its resources is created, it's time to run the deployment. This code is broken up into smaller sections to emphasize different parts of its logic.


```go
func createDeployment() (resources.DeploymentExtended, error) {
        template, err := readJSON(templateFile)
        if err != nil {
                return resources.DeploymentExtended{}, err
        }
        params, err := readJSON(parametersFile)
        if err != nil {
                return resources.DeploymentExtended{}, err
        }

        // ...
```

The deployment files are loaded by `readJSON`, the details of which are skipped here. This function returns a `*map[string]interface{}`, the type used in
constructing the metadata for the resource deployment call.

```go
        // ...
        
        deploymentsClient := resources.NewDeploymentsClient(config.SubscriptionID)
        deploymentsClient.Authorizer = autorest.NewBearerAuthorizer(token)

        deploymentFuture, err := deploymentsClient.CreateOrUpdate(
                ctx,
                resourceGroupName,
                deploymentName,
                resources.Deployment{
                        Properties: &resources.DeploymentProperties{
                                Template:   template,
                                Parameters: params,
                                Mode:       resources.Incremental,
                        },
                },
        )
        if err != nil {
                log.Fatalf("Failed to create deployment: %v", err)
        }
        //...
```

This code follows the same pattern as with creating the resource group. A new client is created, given the ability to authenticate with Azure, and then a method is called. 
The method even has the same name (`CreateOrUpdate`) as the corresponding method for resource groups. This pattern is seen again and again in the SDK. 
Methods that perform similar work normally have the same name.

The biggest difference comes in the return value of the `deploymentsClient.CreateOrUpdate()` method. This value is a `Future` object, which follows the 
[future design pattern](https://en.wikipedia.org/wiki/Futures_and_promises). Futures represent a long-running operation in Azure that you may want to 
occasionally poll while performing other work.

```go
        //...
        err = deploymentFuture.Future.WaitForCompletion(ctx, deploymentsClient.BaseClient.Client)
        if err != nil {
                log.Fatalf("Error while waiting for deployment creation: %v", err)
        }
        return deploymentFuture.Result(deploymentsClient)
}
```

For this example, the best thing to do is to wait for the operation to complete. Waiting on a future requires both a [context object](https://blog.golang.org/context) and the client that created
the Future object. There are two possible error sources here: An error caused on the client side when trying to invoke the method, and an error response from the server. The latter is returned as
part of the `deploymentFuture.Result()` call.

### Obtaining the assigned IP address

To do anything with the newly created VM, you need the assigned IP address. IP addresses are their own separate Azure resource, bound to Network Interface Controller (NIC) resources.

```go
func getLogin() {
        params, err := readJSON(parametersFile)
        if err != nil {
                log.Fatalf("Unable to read parameters. Get login information with `az network public-ip list -g %s", resourceGroupName)
        }

        addressClient := network.NewPublicIPAddressesClient(config.SubscriptionID)
        addressClient.Authorizer = autorest.NewBearerAuthorizer(token)
        ipName := (*params)["publicIPAddresses_QuickstartVM_ip_name"].(map[string]interface{})
        ipAddress, err := addressClient.Get(ctx, resourceGroupName, ipName["value"].(string), "")
        if err != nil {
                log.Fatalf("Unable to get IP information. Try using `az network public-ip list -g %s", resourceGroupName)
        }

        vmUser := (*params)["vm_user"].(map[string]interface{})
        vmPass := (*params)["vm_password"].(map[string]interface{})

        log.Printf("Log in with ssh: %s@%s, password: %s",
                vmUser["value"].(string),
                *ipAddress.PublicIPAddressPropertiesFormat.IPAddress,
                vmPass["value"].(string))
}
```

This method relies on information that is stored in the parameters file. The code could query the VM directly to get its NIC, query the NIC to get its IP resource, and then query the IP resource directly. That's a long chain of dependencies and operations to resolve, making it expensive. Since the JSON information is local, it can be loaded instead.

The values for the VM user and password are likewise loaded from the JSON.

## Next steps

In this quickstart, you took an existing template and deployed it through Go. Then you connected 
to the newly created VM via SSH to ensure that it's running.

To continue learning about working with virtual machines in the Azure environment with Go, take a look at the [Azure compute samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/compute) or [Azure resource management samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/resources).
