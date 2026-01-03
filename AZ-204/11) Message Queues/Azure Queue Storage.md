# Azure Queue Storage

When you don't need the feature of Service Bus  e.g. Subscription, topic, filtering and just need a queue to push messages. Azure Queue Storage in a blob storage account can be a cheaper solution.

Characteristic:
- Message size can be 64Kb
- No subscriptions 
- Massive size only limited to storage account 500TB
- Messages can have time to live, but can also set to -1 to never expire. Seven days is the default if the parameter is omitted. 

**URL format:** to access messages in the queue.

```
https://<storage account>.queue.core.windows.net/<queue>
```

## Example
### Send Message

```c#
string queueName = "";

QueueClient queueClient = new QueueClient( 
	new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"), 
	new DefaultAzureCredential()
);

//create queue if it doesn't exist
await queueClient.CreateIfNotExists();

await queueClient.SendMessageAsync("First message"); 
await queueClient.SendMessageAsync("Second message");

SendReceipt receipt = 
await queueClient.SendMessageAsync("Third message");

// Update Third Messaage 
await queueClient.UpdateMessageAsync(
	receipt.MessageId, 
	receipt.PopReceipt, 
	"Third message has been updated"
);
```

### Read Message

```c#
string queueName = "";

QueueClient queueClient = new QueueClient( 
	new Uri($"https://{storageAccountName}.queue.core.windows.net/{queueName}"), 
	new DefaultAzureCredential()
);

// Peek at messages in the queue
PeekedMessage[] peekedMessages = 
await queueClient.PeekMessagesAsync(maxMessages: 10);

foreach (PeekedMessage peekedMessage in peekedMessages)
{
    // Display the message
    Console.WriteLine($"Message: {peekedMessage.MessageText}");
}
```

