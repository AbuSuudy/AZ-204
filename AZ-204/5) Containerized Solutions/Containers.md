# Containers 

## Types of Containers in Azure

### Azure Container Instances 
Azure Container Instances (ACI) provides a single pod of Hyper-V isolated containers on demand. It can be thought of as a lower-level "building block" option compared to Container Apps. Concepts like scale, load balancing, and certificates aren't provided with ACI containers.

The top-level resource in Azure Container Instances is the _container group_

https://learn.microsoft.com/en-us/azure/container-instances/container-instances-container-groups
https://learn.microsoft.com/en-us/azure/container-apps/compare-options


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

### Azure App Service Containers 


### Azure Kubernetes Service 
