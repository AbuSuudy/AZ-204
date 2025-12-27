# Event Hub

Azure Event Hubs is a scalable event processing service that ingests and processes large volumes of events and data, with low latency and high reliability.
## Anatomy of the Event Hub

![](Images/Pasted%20image%2020251225184053.png)

- *Event hub namespace*:  is a collection of event hubs.
- *Event hub*: is a append-only distributed log, which can comprise one or more partitions.  You can configure up to 100 partitions per event hub and 200 in a namespace.
- *Consumer applications*: can consume data by seeking through the event log and maintaining their position. *Each consumer can read one partition* at a time. This allows for consumers to read events in the same event hub in parallel. Consumer group 2 has three instances so reads all three partitions in *parallel* 🔥. Consumer only support a pull model.
- *Consumer group*: This logical group of consumer instances reads data from an event hub. Each consumer group will have all the events in the event hub. This will allow you to invoke different logic on the same event by using separate consumer groups.  
## Publishing vs Consuming 
*Publishers* can publish events using HTTPS or AMQP 1.0 or the Kafka protocol. The choice to use AMQP or HTTPS is specific to the usage scenario. AMQP requires the establishment of a persistent bidirectional socket which has higher network costs when initializing the session, however HTTPS requires extra TLS overhead for every request. `EventHubProducerClient` uses AMQP as its default.

*Consumers* uses AMQP or Kafka to receive events from an event hub. Event Hubs supports only the pull model for consumers to receive events from it. AMQP enables higher throughput and lower latency after the initial connection than pull-based mechanisms such as HTTP GET.
## Pricing Units
The main influence in pricing is the below tiers which determine the amount of compute that the event hub has available.
- Throughput Units (TUs) for the standard tier
- Processing Units (PUs) for the premium tier
- Capacity Units (CUs) for the dedicated tier)

Throughput = `Consumer Available` + `Pricing Unit` + `Number of partition` 
- Where `Consumer Available` ~ `Number of Partitions`
## Partition
A partition is a data organization mechanism that enables parallel publishing and consumption. A partition can be seen as commit log. Partitions hold event data that contains the following information:
- User-defined properties
- Position in the stream sequence
- Service-side timestamp at which it was accepted

![](Images/Pasted%20image%2020251225233210.png)

For tiers other than the premium and dedicated tiers, you can't change the partition count for an event hub after its creation. For an event hub in a premium or dedicated tier, you can increase the partition count after its creation, but you can't decrease them.]
### Partition Key 
It's a best practice for publishers to remain unaware of the specific partitioning model chosen for an event hub and to only specify a partition key that is used to consistently assign related events to the same partition.

![](Images/Pasted%20image%2020251226001606.png)
## Checkpoint
Checkpointing Is a process by which consumer marks their position within a partition event sequence. This is stored in blob store. Checkpointing is the responsibility of the consumer and occurs on a per-partition basis within a consumer group. Offset can be seen a cursor. 

If a reader disconnects from a partition, when it reconnects it begins reading at the checkpoint that was previously submitted by the last reader of that partition in that consumer group.

Users should decide the frequency of updating the checkpoint. Updating after each successfully processed event can have performance and cost implications as it triggers a write operation to the underlying checkpoint store. Also, checkpointing every single event is indicative of a queued messaging pattern for which a Service Bus queue might be a better option than an event hub. The idea behind Event Hubs is that you get "at least once" delivery at great scale.

![400](Images/Pasted%20image%2020251224222427.png)

### Guidance to use Azure Blob Storage as a checkpoint
- Use a separate container for each consumer group.
- Don't use the storage account for anything else.
- Don't use the container for anything else
- Create the storage account in the same region

Disable this functionality: 
- Hierarchical namespace
- Blob soft delete
- Versioning

```
Blob Container
 └── ConsumerGroupName/
      └── PartitionId/
           └── checkpoint.json (offset, sequence number)
```
## Balance Partition Load between Consumers 
The key to scale for Event Hubs is the idea of *partitioned consumer.* In contrast to the competing consumers pattern.

*Competing consumers* - You have multiple consumers (workers) reading from the same queue or subscription. Each consumer competes for messages.  Partitioned consumer are pinned to reading one or many partitions. 

![](Images/Pasted%20image%2020251227161722.png)

When you design a consumer in a distributed environment, the scenario must handle the following requirements:

1) **Scale:** Create multiple consumers, with each consumer taking ownership of reading from a few Event Hubs partitions.
2) **Load balance:** Increase or reduce the consumers dynamically
3) **Seamless resume on failures:** If a consumer (**consumer A**) fails, then other consumers can pick up the partitions owned by another consumer and pick up where it left off.

### Event Processor Client 
*EventProcessorClient*  is intended to provide a robust experience for processing events. Ownership of partitions is evenly distributed among all the active event processor instances within the consumer group. 

An event processor client instance typically owns and processes events from one or more partitions.  

All event processor instances communicate with a central store periodically to update its own processing state and to learn about other active instances. This data is then used to balance the load among the active processors. Change ownership of failed consumers. 
## Event Retention 
You can't explicitly delete events. Published events are removed from an event hub based on a configurable, timed-based retention policy. Events are automatically removed when the retention period has been reached. 

- *Default* and shortest retention is 1 hour
- *Standard* the maximum retention period is 7 days
- *Premium and Dedicated* maximum retention period of 90 days

> [!NOTE] 
> If you change the retention period, it applies to all events including events that are already in the event hub.
## Application Group Policies
An application group is a collection of client applications that connect to an Event Hubs. Azure Event Hubs enables you to define resource access policies such as throttling policies for a given application group and controls event streaming (publishing or consuming) between client applications and Event Hubs.
## Schema Registry in Event Hubs
Azure Schema Registry in Event Hubs provides a centralized repository for managing schemas of event streaming applications.

![](Images/Pasted%20image%2020251225165350.png)
## Storage 
Event Hubs to ingest and store streaming data temporarily, but you can capture your data in near real time in Azure Blob Storage or Azure Data Lake Storage for long-term retention.

![](Images/Pasted%20image%2020251225180549.png)
## Explore streaming data with Azure Data Explorer
You can query event hub using Azure Data Explorer 

![](Images/Pasted%20image%2020251225180411.png)

## Push Batched Event 

```c#
public async Task UpdateChannelEventHub(List<MetasphereChannel> metasphereChannels)
{
	// uses AMQP as default
	 EventHubProducerClient producerClient = new EventHubProducerClient(
		 Environment.GetEnvironmentVariable("EventHub"),
		 "MetasphereApi",
		 new DefaultAzureCredential());
	 
	 var options = new JsonSerializerOptions { WriteIndented = true };
	 
	 EventDataBatch eventBatch = await producerClient.CreateBatchAsync();
	 
	 try
	 {
		foreach (var channel in metasphereChannels)
		{
			var channelData = JsonSerializer.Serialize(channel, options);				
         //Under the hood, Azure Event Hubs speaks AMQP, a binary, framed,
         //transfer protocol. So data is converted to bytes to send over the wire
         
			var bytes = Encoding.UTF8.GetBytes(channelData);
			 
			if (!eventBatch.TryAdd(new EventData(bytes))))
			{
				 await producerClient.SendAsync(eventBatch);
				 
				 eventBatch.Dispose();
				 
				 eventBatch = await producerClient.CreateBatchAsync();
				 
				 eventBatch.TryAdd(new EventData(bytes)));
				 
			}
		}
		 
		 await producerClient.SendAsync(eventBatch);
	}
	finally
	{
		 eventBatch.Dispose();
		 await producerClient.DisposeAsync();
	}
}
```
 
