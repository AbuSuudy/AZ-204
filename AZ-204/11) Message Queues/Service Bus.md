# Service Bus

Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics. Data is transferred between different applications and services using messages. 

Messages are delivered in **pull** mode, only delivering messages when requested. Unlike consistent polling the service bus uses long polling where the connection is made and service bus is empty it will remain open until a configured timeout has been set in case a  new message comes in and you can skip the overhead of establishing another TCP connection.
## Benefits 
- *Load-Balancing*: allow you to use multiple consumers and due to pull based model it can be consumed at the pace the consumer is comfortable with.
- *Topics and Subscriptions*: Routes the same messages to multiple locations 
- *Decouple Application*: so messages could be processed if consumer is back online.
- *Transactions*: allow a chain of actions to takes place. Once a message has been processed by consumer on a queue. It could be placed onto another queue to for another consumer to execute a separate set of tasks.
- *FIFO* - guarantee 
## Protocol 
The primary wire protocol for Service Bus is Advanced Messaging Queueing Protocol (AMQP) 1.0. Similar to what event hub uses on the consumer end. Also uses the same protocol is shared by on-premises brokers such as ActiveMQ or RabbitMQ.
## Queue 
Messages are sent to and received from **queues**. Queues store messages until the receiving application is available to receive and process them.

![](Images/Pasted%20image%2020251228132124.png)
## Topic
Publishers send messages to a topic in the same way that they send messages to a queue. But, consumers don't receive messages directly from the topic. Instead, consumers receive messages from subscriptions of the topic. A topic subscription resembles a virtual queue that receives certain copies of the messages that are sent to the topic.

*Subscriptions* are durable by default, but can be configured to expire and then be automatically deleted. You can have rules on each subscription that filter messages that are received in each topic  and an optional **action** that can modify message metadata.

![](Images/Pasted%20image%2020251228150408.png)
## Filter and Actions
Subscribers can define which messages they want to receive from a topic:
- *Filters* - are conditions that allow certain messages through
- *Actions* - Allows you to add information to an existing message before it enters the subscription queue.  
## Dead Letter Queue
Service Bus queues and topic subscriptions provide a secondary sub queue, called a dead-letter queue (DLQ). The dead letter queue holds messages that can't be delivered to any receiver, or messages that can't be processed.
## Duplicate Deletion 
If any new message is sent with `MessageId` that was logged during the time window, the message is reported as accepted, but the newly sent message is instantly ignored and dropped.   The time window defaults to 10 minutes for queues and topics, with a minimum value of 20 seconds and a maximum value of 7 days.

![](Images/Pasted%20image%2020251228160524.png)
## Transaction 
A **transaction** in Azure Service Bus is a way to group multiple messaging operations so they either **all succeed together or all fail together**.  Ensures transactional integrity for all internal operations.

```c#
var options = new ServiceBusClientOptions { EnableCrossEntityTransactions = true };
await using var client = new ServiceBusClient(connectionString, options);

ServiceBusReceiver receiverA = client.CreateReceiver("queueA");
ServiceBusSender senderB = client.CreateSender("queueB");

ServiceBusReceivedMessage receivedMessage = await receiverA.ReceiveMessageAsync();

using (var ts = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    await receiverA.CompleteMessageAsync(receivedMessage);
    await senderB.SendMessageAsync(new ServiceBusMessage());
    ts.Complete();
}
```
## Message Deferral 
If you read message in service bus but due to some reason you're not ready to process it e.g. if an orders payment hasn't been processed yet before processing the shipping message.  You can defer the message.  Deferred messages aren't expired and automatically moved to a dead-letter queue until a client app attempts to receive them using an API and the sequence number.

For this processed it's the owner is responsible for remembering the **sequence number

```c#
ServiceBusReceivedMessage message = await receiver.ReceiveMessageAsync();

if(ready == false)
{
	await receiver.DeferMessageAsync(message);
	
	//Save Seq number to blob store to retrieve later
	long seq = message.SequenceNumber;
}
```

Get Message using sequence number from blob storage. There could be an azure function that reads deferred message and try and process it based on a timer.

```c#
//Retrieve seq from blob storage
ServiceBusReceivedMessage deferredMessage =
    await receiver.ReceiveDeferredMessageAsync(seq);
  
// if it's still not ready you can add back in the deffer 
if(ready == false)
{
	await receiver.DeferMessageAsync(message);
	
	//Save Seq number to blob store to retrieve later
	long seq = message.SequenceNumber;
}  

```
## Auto Forwarding
Auto‑forwarding allows you to chain a queue or subscription (the source) to another queue or topic (the destination) within the same namespace. 

![](Images/Pasted%20image%2020251228142745.png)

## Choosing between Service Bus, Event Hub and Event Grid 
https://learn.microsoft.com/en-us/azure/service-bus-messaging/compare-messaging-services