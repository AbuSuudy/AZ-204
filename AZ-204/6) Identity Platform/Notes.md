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
### Access Token Flow
The generating of the token is handled when you use this package `Azure.Identity` under the hood it will generate a token based on system assigned / user assigned based on configuration. 

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
### The `DefaultAzureCredential` Chain Order
The configuration is optional above since if credentials are not found it will move on the next on the chain. It can be difficult to debug if creds are created before your ideal type. Also it does slow down the process since you're checking in places where you know you can skip. It's best to be explicit. 

![](Images/Pasted%20image%2020251207014724.png)