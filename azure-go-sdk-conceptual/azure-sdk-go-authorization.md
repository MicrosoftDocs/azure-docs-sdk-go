---
title: Authentication with the Azure SDK for Go
description: Learn about the authentication methods available in the Azure SDK for Go and how to use them.
services: azure
author: sptramer
ms.author: sttramer
manager: carmonm
ms.date: 04/03/2018 
ms.topic: conceptual
ms.prod: azure
ms.technology: azure-sdk-go
ms.devlang: go
ms.service: active-directory
ms.component: authentication
---
# Authentication methods in the Azure SDK for Go

The Azure SDK for Go offers a variety of authentication types and methods that your application can use. Supported authentication methods range from pulling information from environment variables to interactive web-based authentication. This article introduces you to the available types of authentication in the SDK, and the methods for using them. You'll also learn best practices for selecting which authentication type is right for your application.

## Available authentication types and methods

The Azure SDK for Go offers several different types of authentication, using different credentials sets. Each of these authentication types are available through different authentication methods, which are how the SDK takes these credentials as input. The following table describes the available types of authentication and situations in which they're recommended for use by your application.

| Authentication type | Recommended when... |
|---------------------|---------------------|
| Certificate-based authentication | You have an X509 certificate that was configured for an Azure Active Directory (AAD) user or service principal. To learn more, see [Get started with certificate-based authentication in Azure Active Directory]. |
| Client credentials | You have a configured service principal that is set up for this application or a class of applications it belongs to. To learn more, see [Create a service principal with Azure CLI 2.0]. |
| Managed Service Identity (MSI) | Your application is running on an Azure resource that has been configured with Managed Service Identity (MSI). To learn more, see [Managed Service Identity (MSI) for Azure resources]. |
| Device token | Your application is meant to be used interactively __only__ and will have a variety of users, potentially from multiple AAD tenants. Users have access to a web browser to sign in. For more information, see [Use device token authentication](#use-device-token-authentication).|
| Username/password | You have an interactive application that cannot use any other authentication method. Your users do not have multi-factor authentication enabled for their AAD sign in. |

> [!IMPORTANT]
> If you use an authentication type other than client credentials, your application must be registered in Azure Active Directory. To learn how,
> see [Integrating applications with Azure Active Directory](/azure/active-directory/develop/active-directory-integrating-applications).

> [!NOTE]
> Unless you have special requirements, avoid username/password authentication. In situations where user-based sign in is appropriate, device token authentication can usually be used instead.

[Get started with certificate-based authentication in Azure Active Directory]: /azure/active-directory/active-directory-certificate-based-authentication-get-started
[Create a service principal with Azure CLI 2.0]: /cli/azure/create-an-azure-service-principal-azure-cli
[Managed Service Identity (MSI) for Azure resources]: /azure/active-directory/managed-service-identity/overview

These authentication types are available through different methods. [_Environment-based authentication_](#use-environment-based-authentication) reads credentials directly from the program's environment. [_File-based authentication_](#use-file-based-authentication) loads a file containing service principal credentials. [_Client-based authentication_](#use-an-authentication-client) uses an object in Go code and makes you responsible for providing the credentials during program execution. Finally, [_Device token authentication_](#use-device-token-authentication) requires users to sign in interactively through a web browser with a token, and cannot be used with environment- or file-based authentication.

All authentication functions and types are available in the `github.com/Azure/go-autorest/autorest/azure/auth` package.

> [!NOTE]
> Unless you have special requirements, avoid client-based authentication. This method of authentication encourages bad practices. In particular, using client-based authentication makes it tempting to hard-code credentials. Writing custom code for authentication may also break under future SDK releases if authentication requirements change.

## Use environment-based authentication

If you're running your application in a tightly controlled environment such as in a container, environment-based authentication is a natural choice. You configure the shell environment before running your application and the Go SDK reads these environment variables at runtime to authenticate with Azure. 

Environment-based authentication has support for all authentication methods except device tokens, evaluated in the following order: Client credentials, certificates, username/password, and Managed Service Identity (MSI). If a required environment variable is unset or the SDK gets a refusal from the authentication service, the next authentication type is tried. If the SDK cannot authenticate from the environment, it returns an error.

The following table details the environment variables that need to be set for each authentication type supported by environment-based authentication.

| Authentication type | Environment variable | Description |
| ------------------- | -------------------- | ----------- |
| __Client credentials__ | `AZURE_TENANT_ID` | The ID for the Active Directory tenant that the service principal belongs to. |
| | `AZURE_CLIENT_ID` | The name or ID of the service principal. |
| | `AZURE_CLIENT_SECRET` | The secret associated with the service principal. |
| __Certificate__ | `AZURE_TENANT_ID` | The ID for the Active Directory tenant that the certificate is registered with. |
| | `AZURE_CLIENT_ID` | The application client ID associated with the certificate. |
| | `AZURE_CERTIFICATE_PATH` | The path to the client certificate file. |
| | `AZURE_CERTIFICATE_PASSWORD` | The password for the client certificate. |
| __Username/Password__ | `AZURE_TENANT_ID` | The ID for the Active Directory tenant that the user belongs to. |
| | `AZURE_CLIENT_ID` | The application client ID. |
| | `AZURE_USERNAME` | The username to sign in with. |
| | `AZURE_PASSWORD` | The password to sign in with. |
| __MSI__ | | MSI does not require any credentials to be set. The application must be running on an Azure resource configured to use MSI. For details, see [Managed Service Identity (MSI) for Azure resources]. |

If you need to connect to a cloud or management endpoint other than the default Azure public cloud, you can also set the following environment variables. The most common reasons to set them are if you use Azure Stack, a cloud in a different geographic region, or the Azure Classic deployment model.

| Environment variable | Description  |
|----------------------|--------------|
| `AZURE_ENVIRONMENT` | The name of the cloud environment to connect to. |
| `AZURE_AD_RESOURCE` | The Active Directory resource ID to use when connecting. This should be a URI pointing to your management endpoint. |

When using environment-based authentication, call the [NewAuthorizerFromEnvironment](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#NewAuthorizerFromEnvironment) function to get your authorizer object. This object is then set
on the `Authorizer` property of clients to allow them access to Azure.

```go
import "github.com/Azure/go-autorest/autorest/azure/auth"
authorizer, err := auth.NewAuthorizerFromEnvironment()
```

### Authentication on Azure Stack

To authenticate on Azure Stack, you need to set the following variables:

| Environment variable | Description  |
|----------------------|--------------|
| `AZURE_AD_ENDPOINT` | The Active Directory endpoint. |
| `AZURE_AD_RESOURCE` | The Active Directory resource ID. |

These variables can be retrieved from Azure Stack metadata information. To retrieve the metadata, open a web browser in your Azure Stack environment and use the url: `(ResourceManagerURL)/metadata/endpoints?api-version=1.0`

The `ResourceManagerURL` varies based on the region name, machine name and external fully qualified domain name (FQDN) of your Azure Stack deployment:

| Environment | ResourceManagerURL |
|----------------------|--------------|
| Development Kit | `https://management.local.azurestack.external/` |
| Integrated Systems | `https://management.(region).ext-(machine-name).(FQDN)` |

For more details on how to use Azure SDK for Go on Azure Stack see [Use API version profiles with Go in Azure Stack](https://docs.microsoft.com/en-us/azure/azure-stack/user/azure-stack-version-profiles-go)


## Use file-based authentication

File-based authentication only works with client credentials when they are stored in a local file format generated by [the Azure CLI 2.0](/cli/azure). You can easily create this file when creating a new service principal with the `--sdk-auth` parameter. If you plan on using file-based authentication, make sure that this argument is provided when creating a service principal. Since the CLI prints output to `stdout`, redirect output to a file.

```azurecli
az ad sp create-for-rbac --sdk-auth > azure.auth
```

Set the `AZURE_AUTH_LOCATION` environment variable to where the authorization file is located. This environment variable is read by the application, and the credentials within it are parsed. If you need to select the authorization file at runtime, manipulate the program's environment using the [os.Setenv](https://golang.org/pkg/os/#Setenv) function.

To load the authentication information, call the [NewAuthorizerFromFile](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#NewAuthorizerFromFile) function. Unlike environment-based authorization, file-based authorization requires a resource endpoint.

```go
import "github.com/Azure/go-autorest/autorest/azure/auth"
authorizer, err := NewAuthorizerFromFile(azure.PublicCloud.ResourceManagerEndpoint)
```

For more on using service principals and managing their access permissions, see [Create a service principal with Azure CLI 2.0].

## Use device token authentication

If you want users to sign in interactively, the best way to offer that capability is through device token authentication. This authentication flow passes the user a token to paste into a Microsoft sign-in site, where they then authenticate with an Azure Active Directory (AAD) account. This authentication method supports accounts that have multi-factor authentication enabled, unlike standard username/password authentication.

To use device token authentication, create a [DeviceFlowConfig](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#DeviceFlowConfig) authorizer with the [NewDeviceFlowConfig](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#NewDeviceFlowConfig) function. Call [Authorizer](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#DeviceFlowConfig.Authorizer) on the resulting object to start the authentication process. Device flow authentication blocks program execution until the whole authentication flow is complete.

```go
import "github.com/Azure/go-autorest/autorest/azure/auth"
deviceConfig := auth.NewDeviceFlowConfig(applicationID, tenantID)
authorizer, err := deviceConfig.Authorizer()
```

## Use an authentication client

If you require a specific type of authentication and are willing to have your program do the work to load authentication information from the user, you can use any client that conforms to the [auth.AuthorizerConfig](https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#AuthorizerConfig) interface. Use a type that implements this interface when you want an interactive program, use specialized configuration files, or have a requirement that prevents you from using another authentication method.

> [!WARNING]
> Never hard-code Azure credentials into an application. Putting secrets into an application binary makes it easier for an attacker to
> extract them, whether the application is running or not. This puts all Azure resources the credentials are authorized for at risk!

The following table lists the types in the SDK that conform to the `AuthorizerConfig` interface.

| Authentication type | Authorizer type |
|---------------------|-----------------------|
| Certificate-based authentication | [ClientCertificateConfig] |
| Client credentials | [ClientCredentialsConfig] |
| Managed Service Identity (MSI) | [MSIConfig] |
| Username/password | [UsernamePasswordConfig] |

[ClientCertificateConfig]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#ClientCertificateConfig
[ClientCredentialsConfig]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#ClientCredentialsConfig
[MSIConfig]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#MSIConfig
[DeviceFlowConfig]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#DeviceFlowConfig
[UsernamePasswordConfig]: https://godoc.org/github.com/Azure/go-autorest/autorest/azure/auth#UsernamePasswordConfig

Create an authenticator with its associated `New` function, and then call `Authorize` on the resulting object to perform authentication. For example, to use certificate-based authentication:

```go
import "github.com/Azure/go-autorest/autorest/azure/auth"
certificateAuthorizer := auth.NewClientCertificateConfig(certificatePath, certificatePassword, clientID, tenantID)
authorizerToken, err := certificateAuthorizer.Authorize()
```
