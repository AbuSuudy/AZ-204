# Service Bus

Azure Service Bus is a fully managed enterprise message broker with message queues and publish-subscribe topics. Data is transferred between different applications and services using messages. 

Messages are delivered in **pull** mode, only delivering messages when requested. Unlike consistent polling the service bus uses long polling where the connection is made and service bus is empty it will remain open until a configured timeout has been set in case a  new message comes in and you can skip the overhead of establishing another TCP connection.
## Benefits 
- *Load-Balancing*: allow you to use multiple consumers and due to pull based model it can be consumed at the pace the consumer is comfortable with.
- *Topics and Subscriptions*: Routes the same messages to multiple locations 
- *Decouple Application*: so messages could be processed if consumer is back online.
- *Transactions*: allow a chain of actions to takes place. Once a message has been processed by consumer on a queue. It could be placed onto another queue to for another consumer to execute a separate set of tasks.
## Queue 
Messages are sent to and received from **queues**. Queues store messages until the receiving application is available to receive and process them.

![](Images/Pasted%20image%2020251228132124.png)
## Topic
Publishers send messages to a topic in the same way that they send messages to a queue. But, consumers don't receive messages directly from the topic. Instead, consumers receive messages from subscriptions of the topic. A topic subscription resembles a virtual queue that receives certain copies of the messages that are sent to the topic.

*Subscriptions* are durable by default, but can be configured to expire and then be automatically deleted. You can have rules on each subscription that filter messages that are received in each topic  and an optional **action** that can modify message metadata.

![](Images/Pasted%20image%2020251228140519.png)

https://learn.microsoft.com/en-us/azure/service-bus-messaging/service-bus-messaging-overview#namespaces
## Choosing between Service Bus, Event Hub and Event Grid 
https://learn.microsoft.com/en-us/azure/service-bus-messaging/compare-messaging-services