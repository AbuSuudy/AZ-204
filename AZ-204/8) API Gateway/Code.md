# Code Examples 

I create azure application registration in azure set secrets and use that secret to generate a JWT using the client credential flow. 

```c#
var tenantId = "3853050a-07fd-4295-9fec-e56ecda64939";
var clientId = "45c072b9-bf82-4e02-bd77-f9c7a5c8342a";
var clientSecret = "-tT8Q~FkuT2CSYebspyIYyWtbXXWfvFentxZMaSk";

var authority = $"https://login.microsoftonline.com/{tenantId}/v2.0";

var scopes = new[] { "api://45c072b9-bf82-4e02-bd77-f9c7a5c8342a/.default" };

IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
	.Create(clientId)
	.WithClientSecret(clientSecret)
	.WithAuthority(authority)
	.Build();

AuthenticationResult result = await app
	.AcquireTokenForClient(scopes)
	.ExecuteAsync();

var client = new HttpClient
{
	BaseAddress = new Uri("https://apim-playground-001.azure-api.net")
};

client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

var respose = await client.GetAsync("/FA-Function-Monitoring/HealthCheck");
```

I create a policy on all operation to validate JWT with information 

![](Images/Pasted%20image%2020260313221958.png)

You also disable require subscription id in the settings. You can use subscription key to access all API in API Management and when you use JWT you can limit access by checking claims.

![](Images/Pasted%20image%2020260313222340.png)
## Expose API 
In the application registration you'll need to expose an API to be able to set the audience.

```c#
var scopes = new[] { "api://45c072b9-bf82-4e02-bd77-f9c7a5c8342a/.default" };
```

Will need to look into 

- Delegate permission - Ask user what it need permission for. This will be authorization flow. 
  https://learn.microsoft.com/en-us/entra/identity-platform/howto-update-permissions?pivots=portal 
- Application scopes - Machine to machine you will need to create app roles 
  https://learn.microsoft.com/en-us/entra/identity-platform/howto-add-app-roles-in-apps

![](Images/Pasted%20image%2020260313225444.png)