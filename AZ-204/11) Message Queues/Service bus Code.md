![](Images/Pasted%20image%2020251228150408.png)

## Producer
Producer post messages to a topic. We add a application header `Source` to guide it to the right subscription. 

```C#
ServiceBusClient client = new ServiceBusClient(
	"svcbusns10529.servicebus.windows.net",
    new DefaultAzureCredential()
);

ServiceBusSender sender = client.CreateSender("topic-1");

using ServiceBusMessageBatch messageBatch = await sender.CreateMessageBatchAsync();

for (int i = 1; i <= numOfMessages; i++)
{
    var message = new ServiceBusMessage($"Message {i}")
    {
        ApplicationProperties =
        {
            { "Source", ".NET" }
        }
    };
    
    if (!messageBatch.TryAddMessage(message))
    {
        throw new Exception($"The message {i} is too large to fit in the batch.");
    }
}

await sender.SendMessagesAsync(messageBatch);
```

The subscription in the topic has this filter.
![](Images/Pasted%20image%2020260327181421.png)
## Consumer
The consumer reads from the subscription once it's marked as completed it will be removed from the  subscription.

```c#
ServiceBusProcessor processor = client.CreateProcessor(
	topicName,
	subscription, 
	new ServiceBusProcessorOptions()
);

// add handler to process any errors
processor.ProcessErrorAsync += ErrorHandler;

await processor.StartProcessingAsync();

async Task MessageHandler(ProcessMessageEventArgs args)
{
    string body = args.Message.Body.ToString();
    Console.WriteLine($"Received: {body} from subscription.");
    
    await args.CompleteMessageAsync(args.Message);
}

// handle any errors when receiving messages
Task ErrorHandler(ProcessErrorEventArgs args)
{
    Console.WriteLine(args.Exception.ToString());
    return Task.CompletedTask;
}
```