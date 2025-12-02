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

This is a list of scopes available in the app registration that can be requested from other tenants. Some request could be set to be manually approved by the home tenant admin if it's asking for high privilege scope.

![](Images/Pasted%20image%2020251130160810.png)

During application registration in the home tenant an application registration is created and also  a service principle. If it's multi tenant once an use signs in a service principle is created in that tenant.

![](Images/Pasted%20image%2020251202153318.png)

The reason for multiple service principles in each tenant admin can manage their own permissions. This is stored as **OAuth2PermissionGrants** linked to the Service Principal.
- Tenant A might grant `User.Read`.
- Tenant B might grant `User.Read` + `Mail.Read`.

Here is an example of approval needed from an admin on their local tenant because this flow requires a certain set of scopes.

![](Images/Pasted%20image%2020251202111925.png)

You could also set No admin approval for certain scopes and let the use accept them on their end.

![](Images/Pasted%20image%2020251130160810.png)
## Types of Permissions  

**Delegated Access** - The client application access the resource on behalf of the user.  The application will generate token on behalf of user if they approve and access the resource. The user will be need to have RBAC role access to resource.

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
- Required MFA
- Allow access from specific locations 
- Requiring organization-managed devices for specific applications 