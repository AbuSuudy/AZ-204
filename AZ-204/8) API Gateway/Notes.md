# API Gateway
Acts as reverse proxy that intercept request and forward them to the correct service and allows for a single point of entry. 

![](Images/Pasted%20image%2020251212175920.png)

Components: 
1) *API Gateway* - Gets client request and forward to relevant backend service
   - Authentication 
   - Enforces rate limiting 
   - Centralised logging
   - Cache 
2) *Management Plane* - Provision and configure API management, import APIs, set up policies. 
3) *Developer Portal* -  One stop shop for developers who interact your API. Hosts  documentation, Swagger, create and manage API keys, analytics on their usage and create account.

## Policies
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