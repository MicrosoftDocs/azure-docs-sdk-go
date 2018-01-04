<!-- 
No header is included here so that this can be used in multiple different sections.
This include is so that we can easily update the required Go version, or suggest different
vendoring.
-->
The Azure SDK for Go is compatible with Go versions 1.7 and later. For environments using 
[Azure Stack Profiles](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-version-profiles), Go version 1.9 is the minimum requirement. 
If you do not have Go available on your system, follow [the Go installation instructions](https://golang.org/doc/install). After installation, make 
sure that your environment variables are properly configured by checking `go env`. 

You can obtain the Azure SDK for Go and its dependencies via `go get`.

```bash
go get -u -d github.com/Azure/azure-sdk-for-go/...
```

> [!WARNING]
> Make sure that you captialize `Azure` in the URL. Otherwise this could cause case-related import problems
> when working with the SDK. You will also need to capialize `Azure` in your import statements.

