## Event Hub
![](Images/Pasted%20image%2020251223234520.png)

- Event hub namespace is a collection of event hub instances.
- Each consumer reads from each partition in parallel.
- Checkpointing are saved per partition within a consumer group.
- Each consumer group will have all the events in the event hub. This will allow you to invoke different logic on the same event.
- *Consumer group* Balance load by adding multiple instances - https://learn.microsoft.com/en-us/azure/event-hubs/event-processor-balance-partition-load#consumer-application
- You can configure up to 100  even hub partitions and 200 in a namespace.
 
