## Event Hub

Articles:
- https://learn.microsoft.com/en-us/azure/event-hubs/event-processor-balance-partition-load#consumer-application
- https://learn.microsoft.com/en-us/azure/event-hubs/event-hubs-features


![](Images/Pasted%20image%2020251224235524.png)

- Event hub namespace is collection of event hubs.
- A event hub can be broken down into partition. Each consumer can read one partition at a time. This allows for consumers to read events in the same event hub in parallel. Consumer group 2 has  three instances so reads all three partitions in parallel. 
- You can configure up to 100 partitions per event hub and 200 in a namespace.
  
- _Checkpointing_ is a process by which readers mark or commit their position within a partition event sequence. This is stored in blob store. Checkpointing is the responsibility of the consumer and occurs on a per-partition basis within a consumer group.
  ![400](Images/Pasted%20image%2020251224222427.png)  
- Each consumer group will have all the events in the event hub. This will allow you to invoke different logic on the same event by using separate consumer groups. 
- *Consumer group* Balance load by adding multiple instances - https://learn.microsoft.com/en-us/azure/event-hubs/event-processor-balance-partition-load#consumer-application


## Push Batched Event 

```c#
 public async Task UpdateChannelEventHub(List<MetasphereChannel> metasphereChannels)
 {
     EventHubProducerClient producerClient = new EventHubProducerClient(
         Environment.GetEnvironmentVariable("EventHub"),
         "MetasphereApi",
         new DefaultAzureCredential()
     );
     
     var options = new JsonSerializerOptions { WriteIndented = true };
     
     EventDataBatch eventBatch = await producerClient.CreateBatchAsync();
     
     try
     {
         foreach (var channel in metasphereChannels)
         {
            var channelData = JsonSerializer.Serialize(channel, options);
             
            if (!eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(channelData))))
            {
				 await producerClient.SendAsync(eventBatch);
				 
				 eventBatch.Dispose();
				 
				 eventBatch = await producerClient.CreateBatchAsync();
				 
				 eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes(siteBatch)));
                 
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
 
