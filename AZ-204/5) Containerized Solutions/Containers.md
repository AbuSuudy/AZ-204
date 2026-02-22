# Containers 

## Types of Containers in Azure

### Azure Container Instances 
Azure Container Instance (ACI) was the first containerised solution in Azure it provides a single pod of Hyper-V isolated containers on demand. It can be thought of as a lower-level "building block" option compared to Container Apps. Concepts like scale, load balancing, and certificates aren't provided with ACI containers. User often interact with ACI via another service. Azure Kubernetes Service can layer orchestration and scale on top of ACI through virtual nodes.

The top-level resource in Azure Container Instances is the _container group_. Container groups can share an external IP.  You must expose the port of the IP address from the container.

![500](Images/Pasted%20image%2020260219225537.png)

You can create azure container instance in the azure cli. The `name` fields is the name of the container group.  

```bash 
DNS_NAME_LABEL=aci-example-$RANDOM

az container create --resource-group myResourceGroup \
    --name mycontainer \
    --image mcr.microsoft.com/azuredocs/aci-helloworld \
    --ports 80 \
    --dns-name-label $DNS_NAME_LABEL --location myLocation \
    --os-type Linux \
    --cpu 1 \
    --memory 1.5 
```
### Azure Container Apps
Optimized to run general purpose containers. Is managed solution that doesn't provide direct access to the Kubernetes API. It provides more features than ACI includes: 
- Powered by: Kubernetes, DARP, KEDA (Kubernetes Event-driven Autoscaling) and envoy.
- Service discovery 
- Traffic splitting 
- Enables event driven applications that supporting scale based on traffic, queue and scale to zero. 
- Azure functions support deployments to azure container apps when you need to run the event driven function in the same environment as your other containers.  [Azure Functions base image repos](https://mcr.microsoft.com/en-us/artifact/mar/azure-functions/dotnet-isolated/tags).

![](Images/Pasted%20image%2020260222145847.png)

### Azure App Service Containers
You can use the base image of the app service as the base container. You can deploy the container to the app service that will have an amount of compute it can get from the app service plan.  

App service (application) will have:  scale rules, deployment slots, configuration etc..

```DockerFile
FROM mcr.microsoft.com/appsvc/dotnetcore:lts

ENV PORT 8080
EXPOSE 8080

ENV ASPNETCORE_URLS "http://*:${PORT}"

ENTRYPOINT ["dotnet", "/defaulthome/hostingstart/hostingstart.dll"]
```

### Azure Kubernetes Service 
