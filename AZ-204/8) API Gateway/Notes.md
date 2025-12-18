# API Gateway
Acts as reverse proxy that intercept request and forward them to the correct service and allows for a single point of entry. You will configure the downstream services to only allow request that has been initiated by APIM to prevent people going around it.

![](Images/Pasted%20image%2020251216204719.png)

## Regions 
Depending on the scheme but it could scale out to number of instances also allows for cross region deployment. When you call APIM is will go via azure traffic manager and it will route you to the closest APIM resources. 

## API Components 
![](Images/Pasted%20image%2020251214221107.png)

Components: 
1) *API Gateway* - Gets client request and forward to relevant backend service
   Tasks:
   - Verifies API keys
   - Enforce usage and rate limit
   - Cache 
   - Logs, traces and metrics
   
2) *Management Plane* - Provision and configure API management, import APIs, set up policies and manage users.
   
3) *Developer Portal* -  One stop shop for developers who interact your API. Hosts  documentation, Swagger, create and manage API keys, analytics on their usage and create account.

## Key Concepts 
*API* - Each API contains a reference to the backend service that implements the API, and its operations map to backend operations.

*Products* -  logical grouping of one or more APIs. You could apply authentication rules and policies as a collective.

*User and Group*  - Users can sign up in the developer portal and each user is in on or more groups. API Management has two built in groups:
- **Developer** -  Developers are granted access to the developer portal and build applications that call the operations of an API.
- **Guests** - Unauthenticated developer portal users, such as prospective customers visiting the developer portal. It's read only mode access where they can view the API and not call it.
 
## Policies
API provider can change the behaviour of an API through configuration Such as :
- Transforming the request as they come and also responses as they come out.
- Rate limiting 
- Authentication and Authorisation 
- Routing 
- Checking Cache
- Logging

```xml
<policies>
  <inbound>
    <!-- statements to be applied to the request go here -->
  </inbound>
  <backend>
    <!-- statements to be applied before the request is forwarded to 
         the backend service go here -->
  </backend>
  <outbound>
    <!-- statements to be applied to the response go here -->
  </outbound>
  <on-error>
    <!-- statements to be applied if there's an error condition go here -->
  </on-error>
</policies>
```

API Management enables you to define policies at the following scopes, presented here from broadest to narrowest:
- Global (all APIs)
- Workspace (all APIs associated with a selected workspace)
- Product (all APIs associated with a selected product)
- API (all operations in an API)
- Operation (a single operation in an API)

![](Images/Pasted%20image%2020251215012622.png)

If you have a policy at the global level and a policy configured for an API, both policies can be applied whenever that particular API is used. API Management allows for deterministic ordering of combined policy statements via the `base` element. The example the global policy takes policy first in the inbound request 

```xml
<policies>
    <inbound>
        <base />
        <set-header name="x-request-context-data" exists-action="override">
            <value>@(context.User.Id)</value>
            <value>@(context.Deployment.Region)</value>
      </set-header>
    </inbound>
</policies>
```

### Multi Region backend 
You're able to deploy gateway to multiple region. Only the gateway component of your API Management instance is replicated to multiple regions.

![](Images/Pasted%20image%2020251218000000.png)

You can have same application deployed to multi region. You can check what region the user is coming from and then route them to the closest resources. 

With `@` you can use policy expression which uses C#.
```xml
<policies>
    <inbound>
        <base />
        <choose>
            <when condition="@("West US".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                <set-backend-service base-url="http://contoso-backend-us.com/" />
            </when>
            <when condition="@("East Asia".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                <set-backend-service base-url="http://contoso-backend-asia.com/" />
            </when>
            <otherwise>
                <set-backend-service base-url="http://contoso-backend-other.com/" />
            </otherwise>
        </choose>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

## Self Hosted Gateway in Subnet
You could deploy a self hosted gateway to VM on a subnet or on prem whilst using Site to Site VPN. This will allow resources in the subnet, peered or on prem to use it to access resource API'S. The gateway will need to outbound to the internet outside the subnet to get updates on policy from the central API management instance. 

![](Images/Pasted%20image%2020251218003736.png)