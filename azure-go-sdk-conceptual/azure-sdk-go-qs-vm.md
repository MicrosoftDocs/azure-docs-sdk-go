---
title: Deploy an Azure virtual machine from Go 
description: Deploy a virtual machine using the Azure SDK for Go.
author: sptramer
ms.author: sttramer
manager: carmonm
ms.date: 04/03/2018
ms.topic: quickstart
ms.prod: azure
ms.technology: azure-sdk-go
ms.service: virtual-machines
ms.devlang: go
---
# Quickstart: Deploy an Azure virtual machine from a template with the Azure SDK for Go

This quickstart focuses on deploying resources from a template with the Azure SDK for Go. Templates are snapshots of all of the resources contained within an [Azure resource group](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview). Along the way, you'll become familiar with the functionality and conventions of the SDK while performing a useful task.

At the end of this quickstart, you have a running VM that you log into with a username and password.

[!INCLUDE [quickstarts-free-trial-note](includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](includes/cloud-shell-try-it.md)]

If you use a local install of the Azure CLI, this quickstart requires CLI version __2.0.28__ or later. Run `az --version` to make sure your CLI install meets this requirement. If you need to install or upgrade, see [Install the Azure CLI 2.0](/cli/azure/install-azure-cli).

## Install the Azure SDK for Go

[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

## Create a service principal

To sign in non-interactively to Azure with an application, you need a service principal. Service principals are part of role-based access control (RBAC), which creates a unique user identity. To create a new service principal with the CLI, run the following command:

```azurecli-interactive
az ad sp create-for-rbac --name az-go-vm-quickstart --sdk-auth > quickstart.auth
```

Set the environment variable `AZURE_AUTH_LOCATION` to be the full path to this file. Then the SDK locates and reads the credentials directly from this file, without you having to make any changes or record information from the service principal.

## Get the code

Get the quickstart code and all of its dependencies with `go get`.

```bash
go get -u -d github.com/azure-samples/azure-sdk-for-go-samples/quickstarts/deploy-vm/...
```

You don't need to make any source code modifications if the `AZURE_AUTH_LOCATION` variable is properly set. When the program runs, it loads all the necessary authentication information from there.

## Running the code

Run the quickstart with the `go run` command.

```bash
cd $GOPATH/src/github.com/azure-samples/azure-sdk-for-go-samples/quickstarts/deploy-vm
go run main.go
```

If there is a failure in the deployment, you get a message indicating that there was an issue, but it may not include enough detail. Using the Azure CLI, get the full details of the deployment failure with the following command:

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

### Variables, constants, and types

Since quickstart is self-contained, it uses global constants and variables.

```go
const (
    resourceGroupName     = "GoVMQuickstart"
    resourceGroupLocation = "eastus"

    deploymentName = "VMDeployQuickstart"
    templateFile   = "vm-quickstart-template.json"
    parametersFile = "vm-quickstart-params.json"
)

// Information loaded from the authorization file to identify the client
type clientInfo struct {
    SubscriptionID string
    VMPassword     string
}

var (
    ctx        = context.Background()
    clientData clientInfo
    authorizer autorest.Authorizer
)
```

Values are declared which give the names of created resources. The location is also specified here, which you can change to see how deployments behave in other datacenters. Not every datacenter has all of the required resources available.

The `clientInfo` type is declared to encapsulate all of the information that must be independently loaded from the authentication file to set up clients in the SDK and set the VM password.

The `templateFile` and `parametersFile` constants point to the files needed for deployment. The `authorizer` will be configured by the Go SDK for authentication, and the `ctx` variable is a [Go context](https://blog.golang.org/context) for the network operations.

### Authentication and initialization

The `init` function sets up authentication. Since authentication is a precondition for everything in the quickstart, it makes sense to have it as part of initialization. It also loads some information needed from the authentication file to configure clients and the VM.

```go
func init() {
    var err error
    authorizer, err = auth.NewAuthorizerFromFile(azure.PublicCloud.ResourceManagerEndpoint)
    if err != nil {
        log.Fatalf("Failed to get OAuth config: %v", err)
    }

    authInfo, err := readJSON(os.Getenv("AZURE_AUTH_LOCATION"))
    clientData.SubscriptionID = (*authInfo)["subscriptionId"].(string)
    clientData.VMPassword = (*authInfo)["clientSecret"].(string)
}
```

First, [auth.NewAuthorizerFromFile](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#NewAuthorizerFromFile) is called to load the authentication information from the file located at `AZURE_AUTH_LOCATION`. Next, this file is loaded manually by the `readJSON` function (omitted here) to pull the two values needed to run the rest of the program: The subscription ID of the client, and the service principal's secret, which is also used for the VM's password.

> [!WARNING]
> To keep the quickstart simple, the service principal password is reused. In production, take care to __never__ reuse a password which gives access to your Azure resources.

### Flow of operations in main()

The `main` function is simple, only indicating the flow of operations and performing error-checking.

```go
func main() {
    group, err := createGroup()
    if err != nil {
        log.Fatalf("failed to create group: %v", err)
    }
    log.Printf("Created group: %v", *group.Name)

    log.Printf("Starting deployment: %s", deploymentName)
    result, err := createDeployment()
    if err != nil {
        log.Fatalf("Failed to deploy: %v", err)
    }
    if result.Name != nil {
        log.Printf("Completed deployment %v: %v", deploymentName, *result.Properties.ProvisioningState)
    } else {
        log.Printf("Completed deployment %v (no data returned to SDK)", deploymentName)
    }
    getLogin()
}
```

The steps that the code runs through are, in order:

* Create the resource group to deploy to (`createGroup`)
* Create the deployment within this group (`createDeployment`)
* Obtain and display login information for the deployed VM (`getLogin`)

### Creating the resource group

The `createGroup` function creates the resource group. Looking at the call flow and arguments demonstrates the way that service interactions are structured in the SDK.

```go
func createGroup() (group resources.Group, err error) {
    groupsClient := resources.NewGroupsClient(clientData.SubscriptionID)
    groupsClient.Authorizer = authorizer

        return groupsClient.CreateOrUpdate(
                ctx,
                resourceGroupName,
                resources.Group{
                        Location: to.StringPtr(resourceGroupLocation)})
}
```

The general flow of interacting with an Azure service is:

* Create the client using the `service.New*Client()` method, where `*` is the resource type of the `service` that you want to interact with. This function always takes a subscription ID.
* Set the authorization method for the client, allowing it to interact with the remote API.
* Make the method call on the client corresponding to the remote API. Service client methods usually take the name of the resource and a metadata object.

The [`to.StringPtr`](https://godoc.org/github.com/Azure/go-autorest/autorest/to#StringPtr) function is used to perform a type conversion here. The parameters for SDK methods almost exclusively take pointers, so convenience methods are provided to make the type conversions easy. See the documentation for the [autorest/to](https://godoc.org/github.com/Azure/go-autorest/autorest/to) module for the complete list of convenience converters and their behavior.

The `groupsClient.CreateOrUpdate` method returns a pointer to a data type representing the resource group. A direct return value of this kind indicates a short-running operation that is meant to be synchronous. In the next section, you'll see an example of a long-running operation and how to interact with it.

### Performing the deployment

Once the resource group is created, it's time to run the deployment. This code is broken up into smaller sections to emphasize different parts of its logic.

```go
func createDeployment() (deployment resources.DeploymentExtended, err error) {
    template, err := readJSON(templateFile)
    if err != nil {
        return
    }
    params, err := readJSON(parametersFile)
    if err != nil {
        return
    }
    (*params)["vm_password"] = map[string]string{
        "value": clientData.VMPassword,
    }
        // ...
```

The deployment files are loaded by `readJSON`, the details of which are skipped here. This function returns a `*map[string]interface{}`, the type used in
constructing the metadata for the resource deployment call. The VM's password is also set manually on the deployment parameters.

```go
        // ...

    deploymentsClient := resources.NewDeploymentsClient(clientData.SubscriptionID)
    deploymentsClient.Authorizer = authorizer

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
        return
    }
```

This code follows the same pattern as creating the resource group. A new client is created, given the ability to authenticate with Azure, and then a method is called.
The method even has the same name (`CreateOrUpdate`) as the corresponding method for resource groups. This pattern is seen throughout the SDK.
Methods that perform similar work normally have the same name.

The biggest difference comes in the return value of the `deploymentsClient.CreateOrUpdate` method. This value is of the [Future](https://godoc.org/github.com/Azure/go-autorest/autorest/azure#Future) type, which follows the [future design pattern](https://en.wikipedia.org/wiki/Futures_and_promises). Futures represent a long-running operation in Azure that you can poll, cancel, or block on their completion.

```go
        //...
    err = deploymentFuture.Future.WaitForCompletion(ctx, deploymentsClient.BaseClient.Client)
    if err != nil {
        return
    }
    deployment, err = deploymentFuture.Result(deploymentsClient)

    // Work around possible bugs or late-stage failures
    if deployment.Name == nil || err != nil {
        deployment, _ = deploymentsClient.Get(ctx, resourceGroupName, deploymentName)
    }
    return
```

For this example, the best thing to do is to wait for the operation to complete. Waiting on a future requires both a [context object](https://blog.golang.org/context) and the client that created
the `Future`. There are two possible error sources here: An error caused on the client side when trying to invoke the method, and an error response from the server. The latter is returned as
part of the `deploymentFuture.Result` call.

Once the deployment information is retrieved, there is a workaround for possible bugs where the deployment information may be empty with a manual call to `deploymentsClient.Get` to ensure that the data is populated.

### Obtaining the assigned IP address

To do anything with the newly created VM, you need the assigned IP address. IP addresses are their own separate Azure resource, bound to Network Interface Controller (NIC) resources.

```go
func getLogin() {
    params, err := readJSON(parametersFile)
    if err != nil {
        log.Fatalf("Unable to read parameters. Get login information with `az network public-ip list -g %s", resourceGroupName)
    }

    addressClient := network.NewPublicIPAddressesClient(clientData.SubscriptionID)
    addressClient.Authorizer = authorizer
    ipName := (*params)["publicIPAddresses_QuickstartVM_ip_name"].(map[string]interface{})
    ipAddress, err := addressClient.Get(ctx, resourceGroupName, ipName["value"].(string), "")
    if err != nil {
        log.Fatalf("Unable to get IP information. Try using `az network public-ip list -g %s", resourceGroupName)
    }

    vmUser := (*params)["vm_user"].(map[string]interface{})

    log.Printf("Log in with ssh: %s@%s, password: %s",
        vmUser["value"].(string),
        *ipAddress.PublicIPAddressPropertiesFormat.IPAddress,
        clientData.VMPassword)
}
```

This method relies on information that is stored in the parameters file. The code could query the VM directly to get its NIC, query the NIC to get its IP resource, and then query the IP resource directly. That's a long chain of dependencies and operations to resolve, making it expensive. Since the JSON information is local, it can be loaded instead.

The value for the VM user is also loaded from the JSON. The VM password was loaded earlier from the authentication file.

## Next steps

In this quickstart, you took an existing template and deployed it through Go. Then you connected to the newly created VM via SSH to ensure that it's running.

To continue learning about working with virtual machines in the Azure environment with Go, take a look at the [Azure compute samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/compute) or [Azure resource management samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/resources).

To learn more about the available authentication methods in the SDK, and which authentication types they support, see [Authentication with the Azure SDK for Go](azure-sdk-go-authorization.md).
