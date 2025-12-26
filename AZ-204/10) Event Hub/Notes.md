## Event Hub

Articles:
- https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-features#event-publishers
- https://learn.microsoft.com/en-us/azure/event-hubs/event-processor-balance-partition-load#consumer-application

Azure Event Hubs is a scalable event processing service that ingests and processes large volumes of events and data, with low latency and high reliability
## Anatomy of the Event Hub

![](Images/Pasted%20image%2020251225184053.png)

- *Event hub namespace*:  is a collection of event hubs.
- *Event hub*: is a append-only distributed log, which can comprise one or more partitions.  You can configure up to 100 partitions per event hub and 200 in a namespace.
- *Consumer applications*: can consume data by seeking through the event log and maintaining their position. *Each consumer can read one partition* at a time. This allows for consumers to read events in the same event hub in parallel. Consumer group 2 has three instances so reads all three partitions in *parallel* 🔥.
- *Consumer group*: This logical group of consumer instances reads data from an event hub. Each consumer group will have all the events in the event hub. This will allow you to invoke different logic on the same event by using separate consumer groups.  

## Partition
A partition is a data organization mechanism that enables parallel publishing and consumption. A partition can be seen as commit log. Partitions hold event data that contains the following information:
- User-defined properties
- Its number in the stream sequence
- Service-side timestamp at which it was accepted

![](Images/Pasted%20image%2020251225233210.png)

For tiers other than the premium and dedicated tiers, you can't change the partition count for an event hub after its creation. For an event hub in a premium or dedicated tier, you can increase the partition count after its creation, but you can't decrease them.

For tiers other than the premium and dedicated tiers, you can't change the partition count for an event hub after its creation:
- Throughput Units (TUs) for the standard tier
- Processing Units (PUs) for the premium tier
- Capacity Units (CUs) for the dedicated tier)

The triangle to tune the rate of processing in the event hub.

![300](Images/Pasted%20image%2020251225235921.png)


### Partition Key 
Specifying a partition key enables keeping related events together in the same partition and in the exact order in which they arrived. If you don't specify a partition key when publishing an event, a round-robin assignment is used.

![](Images/Pasted%20image%2020251226001606.png)
## Checkpoint
Checkpointing Is a process by which consumer marks their position within a partition event sequence. This is stored in blob store. Checkpointing is the responsibility of the consumer and occurs on a per-partition basis within a consumer group.

![400](Images/Pasted%20image%2020251224222427.png)


```
Blob Container
 └── ConsumerGroupName/
      └── PartitionId/
           └── checkpoint.json (offset, sequence number)
```

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
 
