# Identity Platform

![](Images/about-microsoft-identity-platform.svg)

The identity platform consists of: 
- **Application Registration** -  Registering your application in Microsoft Entra establishes a trust relationship between your app and the Microsoft identity platform. This will allow your application to authenticate users. 
  
- **Client SDK -** SDK provided by Microsoft to integrate authentication and token acquisition into your applications using OAuth 2.0 and OpenID Connect flows.  Also single sign on which allows you  to use multiple service in Microsoft without needing to re-login. They're all use the same authentication session. You could use the SDK to customise your login experience. 

```c#
using Microsoft.Identity.Client;

//Details from Application Registration
string clientId = "<YOUR_CLIENT_ID>";
string tenantId = "<YOUR_TENANT_ID>";
string redirectUri = "https://localhost:5001/signin-oidc";

string authority = $"https://login.microsoftonline.com/{tenantId}";

// Create Public Client for interactive login
var app = PublicClientApplicationBuilder.Create(clientId)
	.WithRedirectUri(redirectUri)
	.WithAuthority(authority)
	.Build();

// Scopes for delegated permissions
string[] scopes = new string[] { "User.Read" };

//Give me an access every permission that an admin has already granted to my application for that API.
string[] scopes = new string[] {"/.default"" };

// Acquire token interactively
var result = await app.AcquireTokenInteractive(scopes)
	.ExecuteAsync();

var token = result.AccessToken;
```

Cookies for authentication session:
  
![](Images/Pasted%20image%2020251130150113.png)

- **Endpoint**  - That are used to authentication dance to get token. 
- **Audience** - Depending on the audience you will use different backend services.
## Application Registration 
Registering your application in Microsoft Entra establishes a trust relationship between your app and the Microsoft identity platform. Once you register your application, it gets assigned the `User.Read` scope which allows an app to **sign in the user and read their profile**.

> [!NOTE] 
> A scope is essentially a **string identifier** for a permission or group of permissions.

This is a list of scopes available in the app registration. Some request could be set to be manually approved by the home tenant admin if it's asking for high privilege scope.

![](Images/Pasted%20image%2020251130160810.png)

During application registration in the home tenant an application registration is created and also  a service principle. If it's multi tenant once a user signs in a service principle is created in that tenant.

![](Images/Pasted%20image%2020251202153318.png)

The reason for multiple service principles in each tenant admin can manage their own permissions.
- Tenant A might grant `User.Read`.
- Tenant B might grant `User.Read` + `Mail.Read`.

Here is an example of approval needed from an admin on their local tenant because this flow requires a certain set of scopes.

![](Images/Pasted%20image%2020251202111925.png)

You could also set No admin approval for certain scopes and let the user accept them on their end.
Except if the type is an application permission which is always admin approved. This type of permission will be discussed later.

![](Images/Pasted%20image%2020251130160810.png)
## Types of Permissions  
**Delegated Access** - The client application access the resource on behalf of the user.  The application will generate token on behalf of user if they approve and use it to access the resource. The user will be need to have RBAC role access to resource.

**Delegate permission** - An application that is granted the `user_impersonation` . The application is only able to act **on behalf of a user** rather than as itself.

**App only Access**  : Application acts on its own with no user signed in users. The client app must be granted appropriate application permissions of the resource app it's calling e.g. *Machine to Machine flow*.  The app itself will need to have RBAC access.

**Application Permission** -   are used in the app-only access scenario, without a signed-in user present. Always needs admin approval.

You can configure scope permission in the allowed to have on API Permissions tab.  
- *Delegate*:  Can have admin consent 
- *Application* : Must have admin consent and must be approved by the admin on their local tenant.

![](Images/Pasted%20image%2020251130160810.png)

![](Images/access-scenarios.png)
## Consent 
- **User consent** - A user can authorize an application to access some data at the protected resource, while acting as that user.
	- *Static User Consent* - All permissions are asked ahead of time. Bad user experience and feels suspicious to give all these permissions up in one go. 
	- *Dynamic User Consent* - Only when that permission is about to be used.
- **Administrator consent**  - Only on administrator end and will have to wait for a response 
- **Preauthorization** enables a resource application owner to grant permissions without requiring users to see a consent prompt for the same set of permissions that are preauthorized
## Conditional Access 
Conditional Access is Microsoft's Zero Trust policy engine taking signals from various sources into account when enforcing policy decisions.
Common applied Polices:
- Multifactor authentication
- Allowing only Intune enrolled devices to access specific services
- Restricting user locations and IP range
## Authentication Flow

### Authorisation Code Flow with PKCE (MSAL.js)
Runs on an untrusted client where secrets cannot be stored.  This will using the signed in user identity to access the resource and is seen as *delegates access*.

![](Images/convergence-scenarios-native.svg)
### Client Credential Flow (MSAL.NET)
Application is a machine to machine interaction with a service  that sits in your infrastructure and secrets are safe in these environments.  Once permission is approved by admin it can generate token and use it based on the resource identity in a non interactive way (without open browser) . This is seen as *application only access*

![](Images/convergence-scenarios-client-creds.svg)
## Microsoft Graph  
Microsoft Graph gives you a programmatic way to access nearly everything in Microsoft 365: users, groups, mail, calendars, files (One Drive/ SharePoint), Teams, managed devices, and security insights. You can use rest API to create automations you need such as: creating new users, provisioning devices etc. 

API Playground: https://developer.microsoft.com/en-us/graph/graph-explorer

![](Images/Pasted%20image%2020251206181429.png)
## Service Principles 
There are two types of service principles:
- **Application Service Principal**  - Manually created by registering an application in Microsoft Entra ID. : Uses credentials that must be manually managed, such as a client secret (password) or a certificate.

- **Managed Identities Service Principal** - is an automatically managed identity in Microsoft Entra ID can be assigned to an Azure resource. You don't deal with using and rolling secret key values. This is managed by you by Azure. 
## Managed Identities 
There two types of managed identities: 

- **System Assigned Managed Identity** - One-to-one relationship with the Azure resource. Tied to the Azure resource lifecycle. When the resource is deleted, the managed identity associated with it, is automatically deleted.

```bash
az vm create \
--resource-group myResourceGroup \ 
--name myVM-image win2016datacenter \
--generate-ssh-keys \
--assign-identity \
--admin-username azureuser \
--admin-password myPassword12
```

![](Images/Pasted%20image%2020251207162638.png)

- **User Assigned Managed Identity** - can be shared by multiple resources that need a same set of permissions, but will need to be explicitly deleted.   You create user identity first and assign to resources on creation.

``` bash
# Create the identity first
az identity create \
 --resource-group  myResourceGroup \
 --name myUserAssignedIdentity

# Assign identity during creation
az vm create \
--resource-group <RESOURCE GROUP>\
--name <VM NAME>\
--image UbuntuLTS \
--admin-username azureuser \
--admin-password myPassword12\
--assign-identity <USER ASSIGNED IDENTITY NAME>
```

![](Images/Pasted%20image%2020251207162709.png)
### Access Token Flow
The generating of the token is handled when you use this package `Azure.Identity` under the hood it does http request to from Microsoft Entra ID to get a token based on system assigned / user assigned based on configuration. 

![Credential chain sequence diagram](https://learn.microsoft.com/en-us/dotnet/azure/sdk/media/mermaidjs/chain-sequence.svg)

*User Assigned*
```c#
// uses userAssignedClientId
string userAssignedClientId = "<your managed identity client ID>";
var credential = new DefaultAzureCredential(
    new DefaultAzureCredentialOptions
    {      
        ExcludeEnvironmentCredential = true,
        ExcludeWorkloadIdentityCredential = true,
        ManagedIdentityClientId = userAssignedClientId
    }
);

```

*System Assigned*
``` c#
//For System-Assigned MI, ensure ClientId is NOT set
var credential = new DefaultAzureCredential(
    new DefaultAzureCredentialOptions
    {      
        ExcludeEnvironmentCredential = true,
        ExcludeWorkloadIdentityCredential = true
    }
);
```
### The DefaultAzureCredential Chain Order
If that credential fails to acquire an access token, the next credential in the sequence is attempted, and so on, until an access token is successfully obtained.

| Order | Credential                                                                                                                                           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                            | Enabled by default? | Use Case Environment |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------- | -------------------- |
| 1     | [Environment](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.environmentcredential?view=azure-dotnet&preserve-view=true)                | Reads a collection of [environment variables](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/identity/Azure.Identity/README.md#environment-variables) to determine if an application service principal (application user) is configured for the app. If so, `DefaultAzureCredential` uses these values to authenticate the app to Azure. This method is most often used in server environments but can also be used when developing locally. | Yes                 | Deployed Service     |
| 2     | [Workload Identity](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.workloadidentitycredential?view=azure-dotnet&preserve-view=true)     | If the app is deployed to an Azure host with Workload Identity enabled, authenticate that account.                                                                                                                                                                                                                                                                                                                                                     | Yes                 | Deployed Service     |
| 3     | [Managed Identity](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.managedidentitycredential?view=azure-dotnet&preserve-view=true)       | If the app is deployed to an Azure host with Managed Identity enabled, authenticate the app to Azure using that Managed Identity.                                                                                                                                                                                                                                                                                                                      | Yes                 | Deployed Service     |
| 4     | [Visual Studio](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.visualstudiocredential?view=azure-dotnet&preserve-view=true)             | If the developer authenticated to Azure by logging into Visual Studio, authenticate the app to Azure using that same account.                                                                                                                                                                                                                                                                                                                          | Yes                 | Local Developer tool |
| 5     | [Visual Studio Code](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.visualstudiocodecredential?view=azure-dotnet&preserve-view=true)    | If the developer authenticated via Visual Studio Code's [Azure Resources extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureresourcegroups) and the [Azure.Identity.Broker package](https://www.nuget.org/packages/Azure.Identity.Broker) is installed, authenticate that account.                                                                                                                               | Yes                 | Local Developer tool |
| 6     | [Azure CLI](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.azureclicredential?view=azure-dotnet&preserve-view=true)                     | If the developer authenticated to Azure using Azure CLI's `az login` command, authenticate the app to Azure using that same account.                                                                                                                                                                                                                                                                                                                   | Yes                 | Local Developer tool |
| 7     | [Azure PowerShell](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.azurepowershellcredential?view=azure-dotnet&preserve-view=true)       | If the developer authenticated to Azure using Azure PowerShell's `Connect-AzAccount` cmdlet, authenticate the app to Azure using that same account.                                                                                                                                                                                                                                                                                                    | Yes                 | Local Developer tool |
| 8     | [Azure Developer CLI](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.azuredeveloperclicredential?view=azure-dotnet&preserve-view=true)  | If the developer authenticated to Azure using Azure Developer CLI's `azd auth login` command, authenticate with that account.                                                                                                                                                                                                                                                                                                                          | Yes                 | Local Developer tool |
| 9     | [Interactive browser](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true) | If enabled, interactively authenticate the developer via the current system's default browser.                                                                                                                                                                                                                                                                                                                                                         | No                  | Browser              |
| 10    | [Broker](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.interactivebrowsercredential?view=azure-dotnet&preserve-view=true)              | Authenticates using the default account logged into the OS via a broker. Requires that the [Azure.Identity.Broker package](https://www.nuget.org/packages/Azure.Identity.Broker) is installed.                                                                                                                                                                                                                                                         | Yes                 | Local Developer tool |
```c#
clientBuilder.UseCredential(new DefaultAzureCredential(
	new DefaultAzureCredentialOptions
	{
		ExcludeEnvironmentCredential = true,
		ExcludeManagedIdentityCredential = true,
		ExcludeWorkloadIdentityCredential = true,
	}));
```

In the above example it will skip credential types will skip: , `EnvironmentCredential`, `ManagedIdentityCredential`, and `WorkloadIdentityCredential` So the next in line will Visual Studio.

![DefaultAzureCredential using Excludes properties](https://learn.microsoft.com/en-us/dotnet/azure/sdk/media/mermaidjs/default-azure-credential-excludes.svg)

The more configuration you put into the advantages dimmish of ease of use. So better solution would be to use *ChainedTokenCredential* which  act as a empty chain to which you add credentials to suit your app's needs.

```c#
clientBuilder.UseCredential(new ChainedTokenCredential(
	new AzurePowerShellCredential(),
	new VisualStudioCredential()));
```

![ChainedTokenCredential | 350](https://learn.microsoft.com/en-us/dotnet/azure/sdk/media/mermaidjs/chained-token-credential-authentication-flow.svg)
### Using Environment variables 
You can set the environment variable `AZURE_TOKEN_CREDENTIALS` to configure what services are used in the tool chain.

| Value  | Chain used (....................................................................................................................................)                                                     |
| ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Dev`  | ![DefaultAzureCredential with AZURE_TOKEN_CREDENTIALS set to 'dev'](https://learn.microsoft.com/en-us/dotnet/azure/sdk/media/mermaidjs/default-azure-credential-environment-variable-development.svg) |
| `Prod` | ![DefaultAzureCredential with AZURE_TOKEN_CREDENTIALS set to 'prod'](https://learn.microsoft.com/en-us/dotnet/azure/sdk/media/mermaidjs/default-azure-credential-environment-variable-production.svg) |
Also if you just want to use one service in particular you could set the below value to  `AZURE_TOKEN_CREDENTIALS`:
- `AzureCliCredential`
- `AzureDeveloperCliCredential`
- `AzurePowerShellCredential`
- `BrokerCredential`
- `EnvironmentCredential`
- `InteractiveBrowserCredential`
- `ManagedIdentityCredential`
- `VisualStudioCredential`
- `VisualStudioCodeCredential`
- `WorkloadIdentityCredential`
### DefaultAzureCredential Guidance 
`DefaultAzureCredential` is undoubtedly the easiest way to get started with the Azure Identity library, but with that convenience comes tradeoffs:
- *Debugging challenge*: Not sure what part of the chain created the token
- *Performance Overhead* : trying multiple credentials instead of directing to your target. 

So it's best to explicit mention what authentication is used by either using `AZURE_TOKEN_CREDENTIALS`, `DefaultAzureCredentialOptions`, `ChainedTokenCredential`

With help for debugging by placing this in your start up class.

```c#
using AzureEventSourceListener listener = new((args, message) =>
{
    if (args is { EventSource.Name: "Azure-Identity" })
    {
        Console.WriteLine(message);
    }
}, EventLevel.LogAlways);
```

![](Images/Pasted%20image%2020251207154032.png)

> [!NOTE] 
> This logging can also be used for multiple  Azure Services:  Service Bus, Event Hub, Cosmos etc..
