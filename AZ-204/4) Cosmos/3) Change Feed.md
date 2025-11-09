# Change Feed 

The change feed in Azure Cosmos DB is a persistent record of changes to a container in the order the changes occur. Change feed support in Azure Cosmos DB works by listening to an Azure Cosmos DB container for any changes. It then outputs the sorted list of documents that were changed in the order in which they were modified.  

Azure Function can have triggers that would respond to changes in the Change feed.

> [!NOTE] 
> Change feed by default doesn't respond to deletes, but this could be configured by using  *Full-Fidelity Change Feed*. Or use soft delete that will update a column on that row and it will be seen as a update.

The change feed processor has four main components: 

1) **The monitored container**:  is the container that has the data that will be updating the change feed.

2) **The Lease Container**: State storage and help coordination across multiple workers. Keeps track the last item processed by the change feed.
   
3) **The Compute Instance** :  The instance that is running that is consuming the change feed e.g. an Azure function.
   
4) **The Delegate**: That is registered in you consumer application that responds to event changes in the consumer feed. This mostly if you have C# application that's not an azure function. If you use an azure function with a cosmos db trigger you will not need this delegate. The function it self acts as the event handler.

The diagram shows two compute instances, and the change feed processor assigns different ranges to each instance to maximize compute distribution.  The combination of all the leases represents the current state of the change feed processor. The lease collection coordination prevent the same message being read twice.

![](Images/Pasted%20image%2020251109150402.png)

