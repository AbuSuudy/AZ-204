# Blob Storage 

An Azure storage account contains all of your Azure Storage data objects: blobs, files, queues, and tables. The storage account provides a unique namespace for your Azure Storage. An endpoint to access each of these locations.

![](Images/Pasted%20image%2020251015140044.png)

## Types of Blobs
Azure Storage offers three types of blob storage:
- **Block Blobs** :  are ideal for storing text or binary files, and for uploading large files efficiently
- **Append blobs**:  are also made up of blocks, but they are optimized for append operations, making them ideal for logging scenarios.
- **Page blobs** are made up of 512-byte pages up to 8 TB in total size and are designed for frequent random read/write operations. Provide the ability to read/write arbitrary ranges of bytes.

## Hierarchy 
Blobs are store din hierarchy in storage accounts. A container organizes a set of blobs, similar to a directory in a file system.
![](Images/image.png)
## Redundancy 
### Availability Zones
Before going over redundancy options you have to know about Availability Zones.  Availability Zones are separated groups of data centres within the same region. Each availability zone has independent power, cooling, and networking infrastructure, so that if one zone experiences an outage, then regional services, capacity, and high availability are supported by the remaining zones. AZ are separated by several kilometres, and usually are within 100 kilometres. Far enough to reduce he likelihood that more than one will be affected by local outages or weather, but close enough for low latency. Not all azure regions have availability zones and you could us this [list](https://learn.microsoft.com/en-us/azure/reliability/regions-list) 

Types of Availability Deployments: 
- **Zone-redundant storage (ZRS)** copies your data synchronously across three or more Azure availability zones in the primary region

- **Zonal deployments**: A zonal resource is deployed to a single, self-selected availability zone. This approach doesn't provide a resiliency benefit, but it helps you to achieve more stringent latency or performance requirements.
### Storage Account Redundancy Options
- **Locally redundant storage (LRS)** replicates the data within your storage accounts to one or more Azure availability zones located in the primary region of your choice.
  
  ![](Images/Pasted%20image%2020251016120406.png)
- **Geo-redundant storage (GRS)** copies your data synchronously to one or more availability zones in the primary region using LRS to a to a secondary region.

![](Images/Pasted%20image%2020251016123212.png)

- **Zone-redundant storage (ZRS)** copies your data synchronously across three or more Azure availability zones in the primary region.

![](Images/Pasted%20image%2020251016120415.png)

**Geo-zone-redundant storage (GZRS)**  - combines the high availability provided by redundancy across availability zones with protection from regional outages provided by geo-replication.

![](Images/Pasted%20image%2020251016123625.png)

- **Read-access geo-redundant storage (RA-GRS)/  Read-access geo-zone-redundant storage (RA-GZRS)**  :  
	- (GRS or GZRS) replicates your data to another physical location in the secondary region to protect against regional outages.
	- Data in the secondary region isn't directly accessible to users or applications when an outage occurs in the primary region, unless a failover occurs.
	- The failover process updates the DNS entry provided by Azure Storage so that the storage service endpoints in the secondary region become the new primary endpoints for your storage account.
	- RA-GRS/ RA-GZRS - During the failover process you don't have access to the account. It allows the application to use secondary region to have read access until fail over has complete.

## Blob Storage Plans 

| Type of storage account     | Supported storage services                                                                 | Redundancy options                               | Usage                                                                                                                                                                                                                                        |
| --------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Standard general-purpose v2 | Blob Storage (including Data Lake Storage1), Queue Storage, Table Storage, and Azure Files | LRS / GRS / RA-GRS<br>  <br>ZRS / GZRS / RA-GZRS | Standard storage account type for blobs, file shares, queues, and tables. Recommended for most scenarios using Azure Storage. If you want support for network file system (NFS) in Azure Files, use the premium file shares account type.    |
| Premium block blobs         | Blob Storage (including Data Lake Storage1)                                                | LRS  <br>  <br>ZRS                               | Premium storage account type for block blobs and append blobs. Recommended for scenarios with high transaction rates or that use smaller objects or require consistently low storage latency.                                                |
| Premium file shares         | Azure Files                                                                                | LRS  <br>  <br>ZRS                               | Premium storage account type for file shares only. Recommended for enterprise or high-performance scale applications. Use this account type if you want a storage account that supports both Server Message Block (SMB) and NFS file shares. |
| Premium page blobs          | Page blobs only                                                                            | LRS  <br>  <br>ZRS                               | Premium storage account type for page blobs only.                                                                                                                                                                                            |

