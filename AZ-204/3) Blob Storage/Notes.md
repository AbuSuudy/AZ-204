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

![](Images/Pasted%20image%2020251016144414.png)
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
## Access Tiers
In Azure blob storage you pay an early deletion penalty if you move tiers before the minimum retention rate.

| Tier         |                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Hot**      | *Online tier* optimized for storing data that is accessed or modified frequently. The hot tier has the highest storage costs, but the lowest access                                                                                                                                                                                                                                                    |
| **Cool**     | *Online tier* optimized for storing data that is infrequently accessed or modified. Data in the cool tier should be stored for a minimum of **30** days. The cool tier has lower storage costs and higher access costs compared to the                                                                                                                                                                 |
| **Cold**     | *Online tier* optimized for storing data that is rarely accessed or modified, but still requires fast retrieval. Data in the cold tier should be stored for a minimum of **90** days. The cold tier has lower storage costs and higher                                                                                                                                                                 |
| **Archieve** | A *Offline tier* so not accessible for download or via `blobstorage` client. Optimized for storing data that is rarely accessed, and that has flexible latency requirements, on the order of hours. Data in the archive tier should be stored for a minimum of 180 days. If you change access tiers from archive to another tier the file would need to be rehydrated which could take up to 15 hours. |

You could use Lifecyle management on the storage account and create rules which would updated the access tier of the file based on usage. You could set filters on the rule so it could be applied to certain files  

![](Images/Pasted%20image%2020251016152612.png)

> [!NOTE] 
> In premium you don't have access tiers since you'll be getting the max performance available 
## Security 

### SAS Tokens
**SAS (shared access signatures) Token**  - gives granular temporary access to Azure resources.  For example:
- What resource the client has access to 
- What permission they have on these resource
- How long they should have access for

It's useful to grant access to azure resources who's identity doesn't exist in our entra id tenant e.g. external application running on on-prem servers, giving access to file outside of your org.

![](Images/sas-url-token.png)

Depending on the type of token you'll use different keys to generate the SAS token.  You never want to give the account key it would give carte blanche access to the entire storage account for indefinite amount of time until the key get rotated. This is why you use SAS token instead.

There are three types of access SAS signature:
- **User Delegation SAS** -  Token signed with user delegation key that is obtained form Entra Id. The client who uses the URL doesn't have to authenticated. When creating this SAS token the person creating should at least be a `Storage Blob Contributor`
  
- **Service SAS** -  secured with the storage account key.  Delegates access to a resource in **only one** of the Azure Storage services: Blob storage , Queue storage, Table storage, or Azure Files.
  
- **Account SAS** - secured with the storage account key.  Delegates access to resources in one or more of the storage services.

> [!NOTE]
> No imposed maximum time limit; however, best practices recommended that you configure an expiration policy to limit the interval and minimize compromise.

 If I give access via a SAS token to a container you're able to programmatically access to each item in the container in c# or view the container in Azure Blob Storage Explorer desktop app. You won't have access it if you open the SAS URL in the browser. If you apply a SAS token to a file you're able to download the file via the browser.

```c#
string sasUrl = ""

BlobContainerClient containerClient = new BlobContainerClient(new Uri(sasUrl));

await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"- {blobItem.Name}");
}
```

![](Images/Pasted%20image%2020251019181742.png)
### SAS Token Hierarchy 
You're able to give SAS token depending on the hierarchy you are on blob storage 

***Storage account level***
![](Images/Pasted%20image%2020251019185844.png)

***Container*** 
![](Images/Pasted%20image%2020251019185946.png)

***File***
![](Images/Pasted%20image%2020251019191221.png)
### Access Policies 
You can create access policy on the container level that have root: permission and valid date range. You can create SAS token that use this access policy. Once the work is done you're able delete the access policy so the SAS token can no longer be used. This is helpful because forcing key rotation to revoke access can be destructive since every SAS token generated with that key will no longer be valid. 

![](Images/Pasted%20image%2020251019184853.png)

![](Images/Pasted%20image%2020251019185008.png)