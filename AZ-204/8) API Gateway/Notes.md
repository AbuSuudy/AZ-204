# API Gateway
Acts as reverse proxy that intercept request and forward them to the correct service and allows for a single point of entry. 

![](Images/Pasted%20image%2020251212175920.png)

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