## Virtual Network
You use a **Virtual Network (VNet)** in Azure because it’s the _foundation_ for controlling traffic flow, securing resources, and building private, isolated environments. You can peer different virtual networks to each other and depending on the configuration you can have access to  their resources and vice verse,.
### System Route
Azure automatically creates system routes and assigns the routes to each subnet in a virtual network. You can't create system routes, and you can't remove system routes, but you can override some system routes with user defined route.

![697](Images/Pasted%20image%2020260409160241.png)

| *Source* | *Address prefixes*                 | *Next hop type* |
| -------- | ---------------------------------- | --------------- |
| Default  | 10.0.0.0/16 (Sub IP Address space) | Virtual network |
| Default  | 10.1.0.0/16                        | VNet peering\|  |
| Default  | 0.0.0.0/0                          | Internet        |
| Default  | 10.0.0.0/8                         | None            |
| Default  | 172.16.0.0/12                      | None            |
| Default  | 192.168.0.0/16                     | None            |
| Default  | 100.64.0.0/10                      | None            |

- **Virtual network**: Routes traffic between address ranges within the address space of a virtual network.
  
- **Virtual network peering**: When you create a virtual network peering between two virtual networks, the system adds a route for each address range within the address space of each virtual network involved in the peering.
  
- **Internet**: Routes traffic specified by the address prefix to the internet
  
- **None**: Traffic routed to the **None** next hop type is dropped rather than routed outside the subnet.
### User Defined Route
To customize your traffic routes, you shouldn't modify the default routes. You should create custom or user-defined (static) routes, which override the Azure default system routes.

This custom route will force all traffic peered to the hub to go via the WAN firewall.

![](Images/Pasted%20image%2020260409162001.png)

## Network Security Groups
You can use an Azure network security group to filter network traffic between Azure resources in Azure virtual networks. A network security group contains security rules that allow or deny inbound network traffic to, or outbound network traffic. NSG can be used at both subnet and NIC level.
### Inbound Default Rule

**Priority** lower numbers processed before higher numbers because lower numbers have higher priority.

| Priority | Name                          | Source            | Source ports | Destination    | Destination ports | Protocol | Access |
| -------- | ----------------------------- | ----------------- | ------------ | -------------- | ----------------- | -------- | ------ |
| 65000    | AllowVNetInBound              | VirtualNetwork    | 0-65535      | VirtualNetwork | 0-65535           | Any      | Allow  |
| 65500    | AllowAzureLoadBalancerInBound | 0.0.0.0/0         | 0-65535      | 0.0.0.0/0      | 0-65535           | Any      | Deny   |
| 65001    | DenyAllInbound                | AzureLoadBalancer | 0-65535      | 0.0.0.0/0      | 0-65535           | Any      | Allow  |

### Outbound Default Rule

| Priority | Name                  | Source         | Source ports | Destination    | Destination ports | Access | Protocol |
| -------- | --------------------- | -------------- | ------------ | -------------- | ----------------- | ------ | -------- |
| 65000    | AllowVnetOutBound     | VirtualNetwork | 0-65535      | VirtualNetwork | 0-65535           | Allow  | Any      |
| 65001    | AllowInternetOutBound | 0.0.0.0/0      | 0-65535      | Internet       | 0-65535           | Allow  | Any      |
| 65500    | DenyAllOutBound       | 0.0.0.0/0      | 0-65535      | 0.0.0.0/0      | 0-65535           | Deny   | Any      |
### Application Security Group
You group many VM NIC and group them in application security group.  You can now use them in NSG to give those machine access to virtual networks, but without having to deal to with individual machine IP.
## Private Endpoint

If you want azure if you want PaaS service to only  be locked to be used by your resources.
1) *Create Virtual Network* so you can control traffic and isolation of your resource.
2) *Create Private Endpoint* for your resource. It will get private IP from the subnet you're deploying to. 
3) *Link Private DNS Zone to V-NET* so it can resolve the resource to the new private IP locally. The DNS zone will need to be linked to every VNET that needs to resolve that service.
4) *Turn off public access* for the resources so it's only accessible from your own subnet.

![](Images/Pasted%20image%2020260408214056.png)
## Network Security Perimeter
Network security perimeter helps you control *public network access* to resources like Azure PaaS by establishing a secure perimeter. 

- Enable explicit access rules to grant access outside the secure perimeter. That is shared by resources. Instead of configuring each resources firewall individually. 
- Explicit destination of where data can leave to *prevent data exfiltration*
- Resources in perimeter have access to each other out the box.
- Allow private endpoint traffic without the need for explicit access rules.
- Allow lock down on resource where it's infeasible to deploy resource to V-NET and won't be able to use private endpoints.

![](Images/Pasted%20image%2020260409144109.png)