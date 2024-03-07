Novo Nordisk Microsoft Entra ID Training - Code Kata #2
======================================

This training exercise is a **beginner-level** course on [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) at Novo Nordisk and serves as a starting point for developers looking to onboard [Heimdall](https://medium.com/@ZaradarTR/wtf-is-heimdall-646843ec18d0).

## Getting started
These instructions will help you prepare for the kata and ensure that your training machine has the tools installed you will need to complete the assignment(s). If you find yourself in a situation where one or more tools might not be available for your training environment please reach out to your instructor for assistance on how to proceed, post an [issue in our repository](https://github.com/NovoNordisk-OpenSource/dojo/issues) or fix it yourself and update the kata via a [pull request](https://github.com/NovoNordisk-OpenSource/dojo/pulls).

### Prerequisites
* [Azure](https://portal.azure.com/)
* [Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli#install)
* [Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [Azure Functions Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-csharp?tabs=linux%2Cazure-cli#install-the-azure-functions-core-tools)

## Exercise
In this code kata, you will implement a custom claims provider for Microsoft Entra identity platform. The custom claims provider will fetch additional user attributes from external systems via REST API and add them to authentication tokens issued to applications. This kata will help you understand how to integrate external user data into authentication flows and enhance application functionality based on custom claims.

### 1. Login to Azure portal
To get started we need to login to Azure to ensure we have permissions to interact with our [Microsoft Entra ID](https://learn.microsoft.com/en-us/entra/identity/) tenant:

```bash
az login -u johndoe@contoso.com -p VerySecret
```

***Note*** <br/>
By default, this command logs in with a user account and will pick up on the active windows user if no values are supplied. [Azure CLI](https://learn.microsoft.com/en-us/cli/azure) will try to launch a web browser to log in interactively. If a web browser is not available, CLI will fall back to device code login. To login with a service principal, specify `--service-principal`.

### 2. Create a kata directory
Once we have our access permissions sorted we can setup a directory for our exercise files. It's pretty straight forward:

```bash
mkdir kata2
cd kata2
```

### 3. Create an Azure functions project
Run the `func init` command, as follows, to create a functions project in a folder named `AzureFunctionsDemo` with the specified runtime:

```bash
func init AzureFunctionsDemo --worker-runtime dotnet-isolated --target-framework net8.0
```

### 4. Step into Azure functions project
Navigate into the project folder:

```bash
cd AzureFunctionsDemo
```

### 5. Create a HttpTrigger function
Add a function to your project by using the following command, where the `--name` argument is the unique name of your function (HttpExample) and the `--template` argument specifies the function's trigger (HTTP):

```bash
func new --name CustomClaimsProvider --template "HTTP trigger" --authlevel "function"
```

### 6. Add claims injection logic to HttpTrigger function
Add logic to inject our custom claim(s) in our newly created function trigger:

```csharp
namespace AzureFunctionsDemo;

using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;
using System.Text.Json;
using System.Text.Json.Serialization;

public class CustomClaimsProviderTrigger(ILogger<CustomClaimsProviderTrigger> logger)
{
    private readonly ILogger<CustomClaimsProviderTrigger> _logger = logger;

    [Function("CustomClaimsProviderTrigger")]
    public static async Task<IActionResult> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req)
    {
        // Fetch request body
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();

        // Deserialize the request body
        dynamic? data = JsonSerializer.Deserialize<Data>(requestBody);

        // Read the correlation ID from the Azure AD  request    
        string? correlationId = data?.data.authenticationContext.correlationId;

        // Claims to return to Azure AD
        ResponseContent response = new();

        response.Data.Actions[0].Claims.CorrelationId = correlationId;
        response.Data.Actions[0].Claims.ApiVersion = "1.0.0";
        response.Data.Actions[0].Claims.DateOfBirth = "01/01/2000";
        response.Data.Actions[0].Claims.CustomRoles.Add("Writer");
        response.Data.Actions[0].Claims.CustomRoles.Add("Editor");

        return new OkObjectResult(response);
    }

    public class ResponseContent{
        [JsonPropertyName("data")]
        public Data Data { get; set; }

        public ResponseContent()
        {
            Data = new Data();
        }
    }

    public class Data{
        [JsonPropertyName("@odata.type")]
        public string ODataType { get; set; }

        [JsonPropertyName("actions")]
        public List<Action> Actions { get; set; }

        public Data()
        {
            ODataType = "microsoft.graph.onTokenIssuanceStartResponseData";
            Actions = [new Action()];
        }
    }

    public class Action{
        [JsonPropertyName("@odata.type")]
        public string ODataType { get; set; }

        [JsonPropertyName("claims")]
        public Claims Claims { get; set; }

        public Action()
        {
            ODataType = "microsoft.graph.tokenIssuanceStart.provideClaimsForToken";
            Claims = new Claims();
        }
    }

    public class Claims{
        [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
        public string? CorrelationId { get; set; }

        [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
        public string? DateOfBirth { get; set; }

        [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
        public string? ApiVersion { get; set; }

        public List<string> CustomRoles { get; set; }

        public Claims()
        {
            CustomRoles = [];
        }
    }
}
```

### 7. Run HttpTrigger function locally to ensure that it works
Run your function by starting the local `Azure Functions` runtime host from the `AzureFunctionsDemo` folder:

```bash
func start
```

### 8. Create HttpTrigger function infrastructure
Create a resource group named `AzureFunctionsDemo-rg` in your chosen region:

```bash
az group create --name AzureFunctionsDemo-rg --location <REGION>
```

Create a general-purpose storage account in your resource group and region:

```bash
az storage account create --name <STORAGE_NAME> --location <REGION> --resource-group AzureFunctionsDemo-rg --sku Standard_LRS --allow-blob-public-access false
```

Create the function app in Azure:

```bash
az functionapp create --resource-group AzureFunctionsDemo-rg --consumption-plan-location <REGION> --runtime dotnet-isolated --functions-version 4 --name <APP_NAME> --storage-account <STORAGE_NAME>
```

### 9. Deploy the function project to Azure
After you've successfully created your function app in Azure, you're now ready to deploy your local functions project by using the `func azure functionapp publish` command:

```bash
func azure functionapp publish <APP_NAME>
```

### 10. Invoke the function on Azure
After you've successfully created your function app in Azure, you're now ready to deploy your local functions project by using the `func azure functionapp publish` command:

```bash
func azure functionapp logstream <APP_NAME>
```

### 11. Clean up resources
 Once your done with this kata use the following command to delete the resource group and all its contained resources to avoid incurring further costs.:

```bash
az group delete --name AzureFunctionsDemo-rg
```

## Want to help make our training material better?
 * Want to **log an issue** or **request a new kata**? Feel free to visit our [GitHub site](https://github.com/NovoNordisk-OpenSource/dojo/issues).
