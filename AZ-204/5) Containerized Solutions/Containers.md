# Containers 

## Types of Containers in Azure

| Container solution                                                                                                                                  | Resource type                                                                                                                                                                                                                                                                                                                                                            |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [Web App for containers](https://learn.microsoft.com/en-us/azure/app-service/quickstart-custom-container?tabs=dotnet&pivots=container-linux-vscode) | App Service is a fully managed service for hosting HTTP-based web apps that have built-in infrastructure maintenance, security patching, scaling, and diagnostic tooling. Runs your container on Microsoft’s proprietary [App Service runtime](https://github.com/Azure-App-Service/dotnetcore), not on Kubernetes, AKS. Scale, orchestration is managed by app service. |
| [Azure Container Instances](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-overview)                               | Managed container instance. Doesn't come with scale or orchestration. Simple solution.                                                                                                                                                                                                                                                                                   |
| [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/overview)                                                             | Managed Kubernetes, but doesn't give you access to Kubernetes API. Kubernetes lite.                                                                                                                                                                                                                                                                                      |
| [Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/aks/what-is-aks#overview-of-aks)                                                 | Managed Kubernetes, the full cluster resides in your subscription, with the cluster configurations and operations within your control and responsibility.                                                                                                                                                                                                                |
### Azure Container Instances 
Azure Container Instance (ACI) was the first containerised solution in Azure it provides a single pod of Hyper-V isolated containers on demand. It can be thought of as a lower-level "building block" option compared to Container Apps. Concepts like scale, load balancing, and certificates aren't provided with ACI containers. User often interact with ACI via another service. Azure Kubernetes Service can layer orchestration and scale on top of ACI through virtual nodes.

The top-level resource in Azure Container Instances is the _container group_. Container groups can share an external IP.  You must expose the port of the IP address from the container.

![](Images/Pasted%20image%2020260222220838.png)

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
Optimized to run general purpose containers. It's a managed Kubernetes solution that doesn't provide direct access to the Kubernetes API. It provides more features than ACI includes:
- Powered by: Kubernetes, DARP, KEDA (Kubernetes Event-driven Autoscaling) and envoy.
- Service discovery 
- Traffic splitting 
- Enables event driven applications that supporting scale based on traffic, queue and scale to zero. 
- Azure functions support deployments to azure container apps when you need to run the event driven function in the same environment as your other containers.  [Azure Functions base image repos](https://mcr.microsoft.com/en-us/artifact/mar/azure-functions/dotnet-isolated/tags).

![](Images/Pasted%20image%2020260222145847.png)
### Azure App Service Containers
Azure App Service provides fully managed hosting for web applications including websites and web APIs. You can deploy these web applications using code or containers. Azure App Service is optimized for web applications. App service (application) will have:  scale rules, deployment slots, configuration etc..

You can use the app service container as the base container. You can deploy the container to the app service that will have an amount of compute it can get from the app service plan.  

```DockerFile
FROM mcr.microsoft.com/appsvc/dotnetcore:lts

ENV PORT 8080
EXPOSE 8080

ENV ASPNETCORE_URLS "http://*:${PORT}"

ENTRYPOINT ["dotnet", "/defaulthome/hostingstart/hostingstart.dll"]
```
### Azure Kubernetes Service 
Azure Kubernetes Service (AKS) provides a fully managed Kubernetes option in Azure. It supports direct access to the Kubernetes API and runs any Kubernetes workload. The full cluster resides in your subscription, with the cluster configurations and operations within your control and responsibility.

