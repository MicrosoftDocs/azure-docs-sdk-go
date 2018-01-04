---
title: Deploy an Azure virtual machine from Go 
description: Deploy a virutal machine using the Azure SDK for Go.
keywords: azure, virtual machine, vm, go, golang, azure sdk
author: sptramer
ms.author: sttramer
ms.date: 10/04/2017
ms.topic: quickstart
ms.devlang: golang
manager: routlaw
---

# Deploy an Azure virtual machine with the Azure SDK for Go

[Azure Resource Manager](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-overview) is the cornerstone of managing your storage and compute resources distributed throughout Azure. It groups these resources together
in a logical container, allowing them to link together easily and be managed as a group. Most importantly from an operations standpoint, it allows for using templates to deploy resources. Once created, resource groups can have their resource information exported as a template, which can then have some values supplied during resource creation via deployment that create distinct, but functionally identical, deployments to the original.

This quickstart focuses on how to deploy an existing template with Go. While it's easy to work with tools like the Azure CLI to create resources from a deployment, this is an easy way to get you familiar with the functionality and conventions of the SDK while still performing a useful and common task. At the end of this quickstart you will have a running VM that you can log into with a username and password.

[!INCLUDE [quickstarts-free-trial-note](includes/quickstarts-free-trial-note.md)]

<!-- Includes the 'Launch Azure Cloud Shell' section -->
[!INCLUDE [cloud-shell-try-it.md](includes/cloud-shell-try-it.md)]

If you choose to install and use the CLI locally, this quickstart requires that you are running the Azure CLI version 2.0.4 or later. Run `az --version` to make sure your CLI install meets this requirement. If you need to install or upgrade, see [Install Azure CLI 2.0](/cli/azure/install-azure-cli).

## Install the Azure SDK for Go 

<!-- Intentionally does not include header -->
[!INCLUDE [azure-sdk-go-get](includes/azure-sdk-go-get.md)]

## Create a service principal

In order to log in non-interactively with an application, you need a service principal. Service principals are part of Role-Based Authentication (RBAC) which create a unique user identity as well as a set of permissions restrictions. In order to create a new VM and its resources, you need to have a service principal authorized to create new Azure resources. To create a new service principal with the CLI, run:

```azurecli-interactive
az ad sp create-for-rbac --name az-go-vm-quickstart
```

__Make sure__ that you record the `appId` and `password` values in the output. These are used in your application to authenticate with Azure.

For more information on creating and managing Service Principals through the Azure CLI 2.0, see [Create an Azure service principal with Azure CLI 2.0](/cli/azure/create-an-azure-service-principal-azure-cli).

## Get the code

You can get the quickstart code, and all of its dependencies, with `go get`:

```bash
go get -u -d github.com/azure-samples/azure-sdk-for-go-samples/_quickstart/deploy-vm/...
```

This code will compile out of the box, but doesn't run correctly until you provide it information about your Azure account and the created service principal. In `main.go` there is a variable, `config`, which contains an `authInfo` struct. This struct needs to have its field values replaced in order to authenticate correctly.

* `SubscriptionID`: Your subscription ID, which can be obtained from the CLI command `az account show --query id -o tsv`
* `TenantID`: Your tenant ID, which can be obtained from the CLI command `az account show --query tenantId -o tsv`
* `ServicePrincipalID`: The `appId` value recorded when creating the service principal
* `ServicePrincipalSecret`: The `password` value recorded when creating the service principal

There are also values you will need to edit in the `vm-quickstart-params.json` file:

* `vm_password`: The password for the VM user account. This must follow the following password restrictions: It must be at least 6 characters in length, and contain 3 of the following: A lowercase letter, an uppercase letter, a number, or a symbol.
## Running the code

Run the quickstart with the `go run` command.

```bash
cd $GOPATH/src/github.com/azure-samples/azure-sdk-for-go-samples/_quickstart/deploy-vm
go run main.go
```

If there is a failure in the deployment, you will get a message indicating that there was an issue, but without any specific details. Using the Azure CLI, you get the details of
the deployment failure with the following command.

```azurecli
az group deployment show -g GoVMQuickstart -n VMDeployQuickstart
```

If the deployment is successful, you will instead get a message giving the username, IP address, and password for logging into the newly created virtual machine. SSH into this machine to confirm that it is up and running.

## Cleaning up

The easiest way to clean up resources created as part of this quickstart is by deleting the resource group. This is done with the following CLI command.

```azurecli
az group delete -y -n GoVMQuickstart
```
## What the code does

What the quickstart code does is broken down into a block of variables and several small functions, each of which are discussed here.

### Variable assignments and structs

Since this is a self-contained quickstart, leading off with global variables to avoid passing them around, reading environment variables, or taking command-line arguments is fine.

```golang
type authInfo struct {
        TenantID               string
        SubscriptionID         string
        ServicePrincipalID     string
        ServicePrincipalSecret string
}
```

To help with identifying which values need to be filled in by the end user - and could be abstracted out to become command-line arguments or environment variables - the `authInfo` struct is declared. This contains all of the information needed for authorization with the Azure services.

```golang
var (
        //...

        resourceGroupName     = "GoVMQuickstart"
        resourceGroupLocation = "eastus"

        deploymentName = "VMDeployQuickstart"
        templateFile   = "vm-quickstart-template.json"
        parametersFile = "vm-quickstart-params.json"

        token *adal.ServicePrincipalToken
)
```
On the variable side, values are declared which indicate the names of created resources. The location is also specified here, which you can change to see how deployments behave in other datacenters. Not every datacenter has all of the required resources available, so `eastus` was chosen due to its guarantee of having compute resources free.

The `templateFile` and `parametersFile` point to the files which need to be loaded for the deployment. These would be good candidates for taking as command line arguments if you plan on extending this quickstart to make it more like production code.

The service principal token will be covered later.

### init() and authorization

The `init()` method for the code sets up authorization. This could be handled in `main()`, but because authorization is a precondition for _all_ operations in the quickstart working, it makes sense to have it as part of initialization. 

```golang
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

* It gets OAuth configuration information for the `TenantID` provided, by interfacing with Azure Active Directory. The `azure.PublicCloud` object contains endpoints used in the standard Azure configuration. If you are using another cloud, whether for a different region or an on-premises [Azure Stack](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-poc), a different object is used here.
* The `adal.NewServicePrincipalToken()` function is called. This takes the OAuth information along with the service principal login, as well as which style of Azure management is being used (unless you have very specific requiremenets and know what you're doing, this should always be `.ResourceManagerEndpoint` for your cloud.)

This `init()` method on its own can easily be taken out of the quickstart and placed in any Go code which needs to authorize with Azure via service principals. 

### Flow of operations in main()

The `main()` function is simple, only indicating the flow of operations and performing error-checking.

```golang
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

The steps that the code will run through are, in order:

* Create the resource group to deploy to (`createGroup()`)
* Create the deployment within this group (`createDeployment()`)
* Obtain and display login information for the deployed VM (`getLogin()`)

### Creating the resource group

The `createGroup()` function creates the resource group, and is our introduction to the first time the SDK is used to interact with an Azure service on behalf of the user. It demonstrates the way that all service interactions are structured.

```golang
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

* Create the client using the `service.NewXClient()`  method, where `X` is the resource type of the `service` that you want to interact wtih. This always takes a subscription ID.
* Set the authorization method for the client, to allow it to interact with the remote API. In most cases this will be an authorizer created with a token generated by using a service
principal over OAuth.
* Make the function call on the client corresponding to the remote API. These often take the name of the resource being constructed and a metadata object that defines the parameters
to use during creation.

Note that the `to.StringPtr()` function is used to perform a type conversion here. The parameters structs for methods of the SDK almost exclusively take pointers, so these methods are provided to make the type conversions easy. See the documentation for the [autorest.to]() module for the complete list and behavor of convenience converters.

Take note that in this case, the `CreateOrUpdate()` operation returns a pointer to a struct. This indicates a short-running operation that is meant to be synchronous, which does not apply to every method in the SDK. In the next section, you'll see an example of a long-running operation and how to interact with them.

Note that the function is called `createOrUpdate()`, which indicates that creation does not _necessarily_ fail if the group already exists. Instead, if possible, Azure tries to _update_ the settings on the resource.

### Performing the deployment

Once the group to contain its resources is created, it's time to run the deployment. This code will be broken up into two chunks, to make it a little easier to discuss.

```golang
func createDeployment() (resources.DeploymentExtended, error) {
        deploymentsClient := resources.NewDeploymentsClient(config.SubscriptionID)
        deploymentsClient.Authorizer = autorest.NewBearerAuthorizer(token)

        template, err := readJSON(templateFile)
        if err != nil {
                return resources.DeploymentExtended{}, err
        }
        params, err := readJSON(parametersFile)
        if err != nil {
                return resources.DeploymentExtended{}, err
        }

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

We'll start with the `readJSON` function. It just loads up the JSON contained within the specified file, so the details are skipped here. The important element is that it returns a `*map[string]interface{}`, the value taken by both the `Template` and `Parameters` fields of the `DeploymentProperties` struct. Again, structs representing properties of Azure services almost exclusively take pointers.

Although this looks more complicated than creating the resource group, it follows the same pattern: A new client is created, it is given the ability to authenticate with Azure, and then a method is called. The method even has the same name (`CreateOrUpdate`) as the corresponding method for resource groups. This is a pattern that you will see again and again in the SDK. Methods which perform similar work tend to have the same name.

The biggest difference comes in the return value of the `CreateOrUpdate()` method. This is a `Future` object, which follows the [future design pattern](https://en.wikipedia.org/wiki/Futures_and_promises) and represents a long-running operation in Azure that you may want to occasionally poll while performing other work.

```golang
        //...
        err = deploymentFuture.Future.WaitForCompletion(ctx, deploymentsClient.BaseClient.Client)
        if err != nil {
                log.Fatalf("Error while waiting for deployment creation: %v", err)
        }
        return deploymentFuture.Result(deploymentsClient)
}
```

For our purposes, the best thing to do is to wait for the operation to complete since there is no other work to perform. Waiting on a future requires both a [context object](https://blog.golang.org/context) and the client which is responsible for managing the operation represented by the Future itself. Note that even with an error possibly occuring while waiting for completion, this is _distinct_ from the error which could be a natural part of resolving the operation on the server side, which is returned as part of the `Result()` call.

### Obtaining the assigned IP address

To do anything with the newly-created VM, you are going to need the assigned IP address. This is not part of the VM itself. Instead, IP addresses are their own separate resources within Azure, which are then bound to NIC (network interface connections) that tie the virtual machine using them to a network.

```golang
func getLogin() {
        params, err := readJSON(parametersFile)
        if err != nil {
                log.Fatalf("Unable to read parameters. Get login information with `az network public-ip list -g %s", resourceGroupName)
        }

        addressClient := network.NewPublicIPAddressesClient(config.SubscriptionID)
        addressClient.Authorizer = autorest.NewBearerAuthorizer(token)
        ipName := (*params)["publicIPAddresses_QuickstartVM_ip_name"].(map[string]interface{})
        ipAddress, err := addressClient.Get(resourceGroupName, ipName["value"].(string), "")
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

This method relies on information that is stored in the parameters file. We could have also queried the VM directly to get its NIC, and then queried the NIC to find which IP resource it's associated with, and then queried that. But that's a long chain of dependencies and operations to resolve, making it expensive compared to a lookup of a value contained within local data. Since the JSON information is not loaded into an annotated struct, it needs to be typecast to get the string values to display. Fortunately, there is no need for a type switch since we know the exact format of the JSON file itself.

You might notice that the signature for `Get()` is different in this case than earlier ones, in that it does not take a `Context` object and isn't a long-running operation. This is because this method was generated from an earlier version of [go-autorest](https://github.com/azure/go-autorest), the library which provides the generated code that allows for operation with services. Until the GA release of the SDK, you may encounter other methods like these. The [reference documentation](https://godoc.org/github.com/Azure/azure-sdk-for-go) will help guide you. Long-running operations built off of older generated code also _do not_ use `Future` objects. Instead they rely on channels. Be aware of function return types!

## Next steps

In this quickstart, you've taken an existing template and deployed it through Go. Then you've connected 
to the newly created VM via SSH to ensure that it's up and operational.

To continue learning about working with virtual machines in the Azure environment with Go, take a look at some of the available [Azure compute samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/compute) or [Azure resource management samples for Go](https://github.com/Azure-Samples/azure-sdk-for-go-samples/tree/master/resources).
