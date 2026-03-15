# Code Examples 

## Generate Token from Application Registration
I create azure application registration in azure set secrets and use that secret to generate a JWT using the client credential flow. 

```c#
var tenantId = "";
var clientId = "";
var clientSecret = "";

var authority = $"https://login.microsoftonline.com/{tenantId}/v2.0";

var scopes = new[] { $"api://{clientId}/.default" };

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
## Enable JWT Validation on all endpoint in APIM
I create a policy on all operation to validate JWT with information 

![](Images/Pasted%20image%2020260313221958.png)

You also disable require subscription id in the settings. You can use subscription key to access all API in API Management and when you use JWT you can limit access by checking claims.

![](Images/Pasted%20image%2020260313222340.png)
## Expose API 
In the application registration you'll need to expose an API to be able to set the audience.

```c#
var scopes = new[] { "api://45c072b9-bf82-4e02-bd77-f9c7a5c8342a/.default" };
```

![](Images/Pasted%20image%2020260313225444.png)

- *Delegate permission* - Ask user what it need permission it needs. What action can delegated to the application service principle. For example, to add an email notification feature to your application, it needs to access your user’s emails. To do so, you would need to request access for the `Mail.ReadWrite` permission.  User will need to present to approve so won't be used for Machine to Machine communication.

- *Application permission* - Used for machine to machine you create JWT token for an application that has set roles claims in JWT. This can be used on the API to role based access if the JWT container certain claims. 

## Creating Role for Claims

![](Images/Pasted%20image%2020260315002930.png)

Assign new role to app registration 
![](Images/Pasted%20image%2020260315003152.png)

![](Images/Pasted%20image%2020260315003331.png)

You grant permission for application registration to use that scope
![](Images/Pasted%20image%2020260315003728.png)

The roles is now included in JWT
```json
{
  "aud": "api://45c072b9-bf82-4e02-bd77-f9c7a5c8342a",
  "iss": "https://sts.windows.net/3853050a-07fd-4295-9fec-e56ecda64939/",
  "iat": 1773533939,
  "nbf": 1773533939,
  "exp": 1773537839,
  "aio": "ASQA2/8bAAAAR1Y2Qa/K5DHT31EN8EiP0tRR4J911edoEgfG+tweULE=",
  "appid": "45c072b9-bf82-4e02-bd77-f9c7a5c8342a",
  "appidacr": "1",
  "idp": "https://sts.windows.net/3853050a-07fd-4295-9fec-e56ecda64939/",
  "oid": "9b28ca89-5cb7-418f-adce-fe786c6d67d0",
  "rh": "1.AQwACgVTOP0HlUKf7OVuzaZJOblywEWCvwJOvXf5x6XINCoAAAAMAA.",
  "roles": [
   "APIM.Admin"
  ],
  "sub": "9b28ca89-5cb7-418f-adce-fe786c6d67d0",
  "tid": "3853050a-07fd-4295-9fec-e56ecda64939",
  "uti": "dspZJwFuQUSExRcufhMtAA",
  "ver": "1.0",
  "xms_ftd": "DkQvXSkHrqjSmpDKPmSbAlEmlmDNI9l5RGwg2mhSrIsBZnJhbmNlYy1kc21z"
}
```

You could do role based authentication on the API.
```c#
[Authorize(Roles = "APIM.Admin")]
[HttpGet("admin-only")]
public IActionResult GetAdminData()
{
    return Ok("You are an admin!");
}
```
