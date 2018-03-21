---
title: Azure SDK for Go samples for authentication and AAD 
description: Selected samples for working with Azure Active Directory (AAD) and authentication from the Azure SDK for Go.
author: sptramer
ms.author: sttramer
ms.date: 03/21/2018
ms.topic: article
ms.devlang: go
manager: carmonm
---

# Azure SDK for Go samples for authentication and AAD

The following table links to selected samples of Go source code that you can use to authenticate with the Azure SDK for Go and work with Azure Activce Directory (AAD) services. To get all of the sample code and tests for authentication and AAD, use the `go get` command:

```bash
go get -d -u github.com/azure-samples/azure-sdk-for-go-samples/iam
go get -d -u github.com/azure-samples/azure-sdk-for-go-samples/authorization
go get -d -u github.com/azure-samples/azure-sdk-for-go-samples/graphrbac
```

All samples for the Azure SDK for Go are available on [GitHub](https://github.com/Azure-Samples/azure-sdk-for-go-samples).

| Name | Description |
|------|-------------|
| [iam/oauth](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/iam/oauth.go) | How to authenticate with Azure to use services. |
| [authorization/auth](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/authorization/auth.go) | Add, remove, and inspect AAD roles. |
| [graphrbac/graph](https://github.com/Azure-Samples/azure-sdk-for-go-samples/blob/master/graphrbac/graph.go) | Inspect and create service principals and AAD applications. Add secrets to an existing service principal or application. |