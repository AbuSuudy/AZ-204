# Cache and CDN

## Cache 
We can have a in memory cache in each instance we're running, but if it scales to 0 this will be cleared. Also when we scale out we will have a separate in memory cache in each node which leads to inconsistencies. This is when having a distributed cache that can be shared by all instance. Azure have two version of Redis services:
- *Azure Cache for Redis* - The original more expensive service, but will be looking to be retired on  September 30, 2028
- *Azure Managed Redis* - The newer version created in collaboration with Redis.
### Fusion Cache
[FusionCache](https://github.com/ZiggyCreatures/FusionCache) is an easy to use, fast and robust hybrid cache with advanced resiliency features. Such as keep all in memory cache in sync , cache stamped protection and may other features. 

- *Factory* -  Get data from data source and set in memory cache.
- *Memory Cache (L1)* - for in memory cache in single node
- *Distributed Cache (L2)* - shared cache instance like Redis
- *Backplane* - in a multi-node scenario, it can notify the other nodes about changes in the cache, so all will be in-sync/

![](Images/Pasted%20image%2020260104154454.png)
## CDN

> [!WARNING] Title
> Azure CDN is being retired and will move to Front Door. Standard and Premium  SKU were on a third party CDN Edgio. It became bankrupt which caused migration.

Microsoft owns CDN network. Entrances to these networks are POP (Point-of-Presence). There are a lot more POP than they're region for resources deployments. In the [UK](https://learn.microsoft.com/en-us/azure/frontdoor/edge-locations-by-region) alone there 5 in London and 1 in Manchester. 

CDN allows you to cache files: `js, HTML, Videos, images etc..` Once cached people in will get a copy of that files from POP closest to their location. Since it stores JavaScript files you can cache a whole client side react react app, but if you're using a framework that uses SSR it won't work since it doesn't have a traditional server to create these pages at request time.

![](Images/Pasted%20image%2020260104152643.png)


