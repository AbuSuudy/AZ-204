## Event Hub

Articles:
- https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-features
- https://learn.microsoft.com/en-us/azure/event-hubs/event-processor-balance-partition-load#consumer-application

Azure Event Hubs is a native data-streaming service in the cloud that can stream millions of events per second, with low latency, from any source to any destination. 
## Anatomy of the Event Hub

![](Images/Pasted%20image%2020251225184053.png)

- *Event hub namespace*:  is a collection of event hubs.
- *Event hub* is a append-only distributed log, which can comprise one or more partitions.  You can configure up to 100 partitions per event hub and 200 in a namespace.
- *Consumer applications*: can consume data by seeking through the event log and maintaining their position. *Each consumer can read one partitio*n at a time. This allows for consumers to read events in the same event hub in parallel. Consumer group 2 has three instances so reads all three partitions in *parallel*.
- *Consumer group*: This logical group of consumer instances reads data from an event hub. Each consumer group will have all the events in the event hub. This will allow you to invoke different logic on the same event by using separate consumer groups.  
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
 
