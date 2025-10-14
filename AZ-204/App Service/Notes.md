# App Service

App service is (PaaS) that host your applications. It abstracts away infrastructure management, letting you focus on your application code.  The *app service* an instance of you application is using resources defined in your *app service plan*. You could have multiple app services sharing resources of an app service plan.

*App Service plan* defines a set of dedicated compute resources for an app to run. Each app service plan defines: 
- Region
- Number and size of VM's
- Operating system used by all VM's
- Pricing Tier 

Pricing tier:

| Category          | Tiers                                                     | Description                                                                                                                                                                                                        |
| ----------------- | --------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Shared compute    | Free, Shared                                              | Run an app on the same Azure VM as other App Service apps, including apps of other customers. One one vm instance. This mostly used for dev/test environment.  *Each app receives a quota of CPU minutes*.<br><br> |
| Dedicated compute | Basic, Standard, Premium, PremiumV2, PremiumV3, PremiumV4 | Run apps on dedicated Azure VMs. The higher the tier, the more VM instances that are available to you for scale-out. The higher the tier, the more VM instances that are available to you for scale-out.<br><br>   |
| Isolated          | IsolatedV2                                                | The same as dedicated compute, but you also have full network isolation by deploying it already created virtual network on azure. So only app in the same VNET can use this app service plan.<br><br>              |

> [!tip] 
> You can start off small and upgrade the app service plan when needed.
## Auto Scaling
Automatic scaling is a *scale out* the which deploys application to other VM using resources from your app service plan. The platform prewarms instances to act as a buffer when scaling out, ensuring smooth performance transitions. The number of vm is based  the compute of your app service plan.

Auto scale set on the app service level per app. So each app can scale independently of each other.

You could configure rule based on how your apps should scale: based on metric (CPU, memory, Usage) or a schedule. Also send email alerts when new instances are created.

> [!NOTE] 
> Rules based auto scale is not available in Free, shared or basic. It's standard and above. Tiers below have manual scale where you select the number of instances you want running.
## Deployment methods
- *Automated* - CI/CD on merges into source control 
- *Manual* - cli, merge into your local git branch, zip deployment and FTP/S
- *Deployment slot*-  live apps with their own host names. You could have use it as staging environment to test Once verified you could deploy staging to prod or roll back the previous prod slot.
## Deployment Slots
Each deployment slot has their hosting and URL so you creating testing, staging environment in the same app.  For dependent resources like database, app insights it would useful to have one for each environment for slots to interact with.

> [!NOTE] 
> You can create separate deployment slots if your plan is on: Standard, Premium or Isolated 

### Swapping Deployment Slot
How to do zero down time deployments using deployment slots. You could reverse the swap to roll back to previous production.

![](App%20Service/Images/Pasted%20image%2020251014132357.png)

### Deployment slot traffic 
You could manage traffic into these deployment slot once if you want to gradual deployment to production.

![](App%20Service/Images/Pasted%20image%2020251014123834.png)

Due to azure deciding which slot you're entering, you provider header information that could direct to your intended slot.

``` 
https://contoso-html.azurewebsites.net/x-ms-routing-name-staging
```

### Deployment Slot Configuration 
Some app setting is deployment slot specific e.g. you want staging to connect to non prod database. If you mark an app setting as a deployment slot setting it doesn't get copied over during a swap, but you're expecting have the same app setting name in prod but with a prod database connection string.

![](App%20Service/Images/Pasted%20image%2020251014132646.png)