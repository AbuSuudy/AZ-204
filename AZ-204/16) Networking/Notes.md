## Virtual Network

Bible: https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview

You use a **Virtual Network (VNet)** in Azure because it’s the _foundation_ for controlling traffic flow, securing resources, and building private, isolated environments. You can peer different virtual networks to each other and depending on the configuration you can have access to  their resources and vice verse,. To filter network traffic between resources in a virtual network, use a network security group or Network Virtual Appliance like firewall.  
### System Route
Azure automatically creates system routes and assigns the routes to each subnet in a virtual network. You can't create system routes, and you can't remove system routes, but you can override some system routes with user defined route.

![697](Images/Pasted%20image%2020260409160241.png)

| *Source* | *Address prefixes*                 | *Next hop type* |
| -------- | ---------------------------------- | --------------- |
| Default  | 10.0.0.0/16 (Sub IP Address space) | Virtual network |
| Default  | 10.1.0.0/16                        | VNet peering    |
| Default  | 0.0.0.0/0                          | Internet        |
| Default  | 10.0.0.0/8                         | None            |
| Default  | 172.16.0.0/12                      | None            |
| Default  | 192.168.0.0/16                     | None            |
| Default  | 100.64.0.0/10                      | None            |
Next Hop:
- **Virtual network**: Routes traffic between address ranges within the address space of a virtual network.
  
- **Internet**: Routes traffic specified by the address prefix to the internet.  If you don't override the Azure default routes, Azure routes traffic for any address not specified by an address range within a virtual network to the internet. . If you don't override the Azure default routes, Azure routes traffic for any address not specified by an address range within a virtual network to the internet.
  
- **None**: Traffic routed to the **None** next hop type is dropped rather than routed outside the subnet. Azure automatically creates default routes for the following address prefixes:
	- **10.0.0.0/8, 172.16.0.0/12, and 192.168.0.0/16**: Reserved for private use in RFC 1918.
	- **100.64.0.0/10**: Reserved in RFC 6598.
	  
	If you assign any of the previous address ranges within the address space of a virtual network, Azure automatically changes the next hop type for the route from **None** to **Virtual network**. 
### Optional Routes
These routes are added when featured are enabled:
- **Virtual network peering**: When you create a virtual network peering between two virtual networks, the system adds a route for each address range within the address space of each virtual network involved in the peering.
  
- `VirtualNetworkServiceEndpoint`: Azure includes public IP addresses of certain services to the route table when you enable a service endpoint. Azure automatically updates these addresses in the route table when service IP addresses change

| Source  | Address prefixes                                        | Next hop type                   |
| ------- | ------------------------------------------------------- | ------------------------------- |
| Default | Unique to the virtual network, for example: 10.1.0.0/16 | VNet peering                    |
| Default | Multiple                                                | `VirtualNetworkServiceEndpoint` |
### User Defined Route
To customize your traffic routes, you shouldn't modify the default routes. You should create custom or user-defined (static) routes, which override the Azure default system routes. 

When you create a route table and associate it to a subnet, the table's routes are combined with the subnet's default routes. If there are conflicting route assignments, UDRs override the default routes.

You can specify the following next hop types when you create a UDR:

- **Virtual Appliance**:  A virtual appliance is a virtual machine that typically runs a network application, such as a firewall. e.g. all traffic will route via the firewall.

|Route Name|Prefix|Next Hop|Purpose|
|---|---|---|---|
|`default-to-fw`|`0.0.0.0/0`|Virtual appliance|Force all outbound through firewall|
  ![](Images/Pasted%20image%2020260409225816.png)

- **Virtual Network Gateway**:  User-defined routes with next hop type Virtual Network Gateway allow you to route traffic to a Virtual Network's gateway which can be paired with network on premises  or another azure V-NET in another tenant. 
  ![](Images/Pasted%20image%2020260410000008.png)

![](Images/Pasted%20image%2020260409223354.png)
![](Images/Pasted%20image%2020260409224111.png)

- **None**: Specify when you want to drop traffic to an address prefix, rather than forwarding the traffic to a destination e.g. if you want to drop all data that will route to the internet.

| Route Name      | Address Prefix | Next Hop Type | Notes                        |
| --------------- | -------------- | ------------- | ---------------------------- |
| `deny-internet` | `0.0.0.0/0`    | `None`        | Blocks all outbound internet |

- **Internet**: Specify the **Internet** option when you want to explicitly route traffic destined to an address prefix to the internet.

https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#how-azure-selects-routes-for-traffic-routing
## Network Security Groups
You can use an Azure network security group to filter network traffic between Azure resources in Azure virtual networks. A network security group contains security rules that allow or deny inbound network traffic to, or outbound network traffic. NSG can be used at both subnet and NIC level.

![](Images/Pasted%20image%2020260409214237.png#center)
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