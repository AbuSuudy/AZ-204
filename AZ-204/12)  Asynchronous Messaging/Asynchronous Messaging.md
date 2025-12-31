# Asynchronous Messaging 

## Types of Messages
### Events
An event is a lightweight notification of a condition or a state change. The publisher of the event has no expectation about how the event is handled. The event data has information about what happened but doesn't have the data that triggered the event. For example, an event notifies consumers that a file was created.
### Commands
A message is raw data produced by a service to be consumed or stored elsewhere. The message contains the data that triggered the message pipeline. The publisher of the message has an expectation about how the consumer handles the message.

The producer *may* wait for response from the consumer once it's done processing. Only after receiving the response does the producer send a new message to start the next operation in the sequence.

![500](Images/Pasted%20image%2020251229171531.png)

## Benefits of using external message stores

- *Load-Balancing*: allow you to use multiple consumers and due to pull based model it can be consumed at the pace the consumer is comfortable with. via the *Competing Consumers pattern*
  
![500](Images/Pasted%20image%2020251229173633.png)

- *Load Levelling* - broker can act as a buffer, and consumers gradually drain messages at their own pace  ![500](Images/Pasted%20image%2020251229174437.png)

- *Fault Tolerant* - If a consumer fails while processing a message, another instance of the consumer can process that message
  
	- *Event Hub*![](Images/Pasted%20image%2020251225184053.png)
	- *Service Bus*  ![](Images/Pasted%20image%2020251228150408.png)
  
- *Decoupling* - Separates the logic of the service producing the message and consuming the message.

- *Offline Support* - Consumers doesn't need to be online e.g.  during deployment messages could be stored in the queue so it can be processed when back online.
## Comparison Between Services
All of these are message broker that implement different messaging patterns.  

| Service     | Direction | Purpose                                                                                                                                                                                                                                  | Type                                 | When to use                                |
| ----------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | ------------------------------------------ |
| Event Grid  | Push      | Pushing event to service or web hook.<br><br>It can handle down stream service being offline. Since it retries to send event in a 24 hour period and adds message to blob storage dead letter queue if it's set up.                      | Event distribution (discrete events) | React to status changes                    |
| Event Hubs  | Pull      | Big data pipeline.  it's a large buffer that's capable of receiving large volumes of data with low latency<br><br><br>Same message can be read by two consumer groups. Whereas service bus a message is owned by one consumer at a time. | Event streaming (series)             | Telemetry and distributed data streaming   |
| Service Bus | Pull      | High-value enterprise messaging that can't be dropped or accept duplicates.                                                                                                                                                              | Message                              | Order processing and financial transaction |

## Patterns

### Competing Consumers 
Multiple consumers might need to compete to read messages from a queue. This pattern explains how to process multiple messages concurrently to optimize throughput

![](Images/Pasted%20image%2020251225184053.png)
### Priority Queue
Some messages are processed before others, this pattern describes how messages posted by a producer with a higher priority are received and processed more quickly by a consumer than messages of a lower priority. You can filter messages in the topic based on priority. Also have separate function that has multiple competing consumer reading messages.

![](Images/Pasted%20image%2020251230110933.png)
## Claim-Check pattern
Instead of sending a large payload over the wire. It will store the data an external storage and give information on how to retrieve the file in the message. This will allow quicker communication due to light weight request data and also helps with going over payload limit to message queues used.
The message size limit for Service Bus premium is 100mb.

This is popular pattern is to used event grid to notify down stream service a blob has been added tot a storage account.

![](Images/Pasted%20image%2020251230144030.png)
### Consideration 
- Create policy in blob storage to clear up files after a certain time has passed.
- Allow for smaller payloads to be transferred as part of the message to prevent the blob storage download overhead when it's not needed.
### Choreography pattern
Using centralised queue where independent services consume a message and execute a task and place the message back into queue but in separate topic for Service B to do their task.

![](Images/Pasted%20image%2020251230112254.png)

#### Issues
If all service are depended on each other they will need to report failures and undo certain actions and this will add complexity.

![350](Images/Pasted%20image%2020251230113550.png)


The workflow can become complicated when choreography needs to occur in a sequence. For instance, Service D can start its operation only after Service B and Service C have completed their operations with success. 

![](Images/Pasted%20image%2020251230143625.png)
### Compensating Transaction pattern
If one or more of the steps fail, you can use the Compensating Transaction pattern to undo the work that the steps performed. Besides performing these steps, the system must also record the counter operations for undoing each step. 

A compensating transaction is an eventually consistent operation itself, so it can also fail. The system should be able to resume the compensating transaction at the point of failure and continue and be idempotent.

The example below is booking flights and the logic in each step in the compensating transaction must take business-specific rules into account. For example, cancelling a flight reservation might not entitle the customer to a complete refund.

![](Images/Pasted%20image%2020251230200449.png)
#### Issues 
- It's not easy to generalize compensation logic

Sample code for tracking changes on a object so that it can be rolled back.

```c#
using System.Runtime.CompilerServices;

namespace AuditTrail
{
    public record AuditEntry(
	    int ChangeOrder, 
	    string PropetyName, 
	    object? OldValue, 
	    object? NewValue,
	    DateTime TimeStamp
    );
    
    public abstract class AuditableEntity
    {
        private List<AuditEntry> _auditLog = new();
        
        private int ChangeOrder { get; set; } = 0;
        
        public IReadOnlyList<AuditEntry> AuditLog => _auditLog.AsReadOnly();
        
        public void SetProperty<T>(ref T field, T value,
         [CallerMemberName] string propertyName = "")
        {
            _auditLog.Add(new AuditEntry(ChangeOrder++, 
            PropetyName: propertyName, OldValue: field, NewValue: value,
            TimeStamp: DateTime.UtcNow));
            
            field = value;
        }
    }
    
    public partial class Person : AuditableEntity
    {
        private string _name;
        public string Name
        {
            get => _name;
            set => SetProperty<string>(ref _name, value);
        }
    }
}
```
### Scheduler Agent Supervisor pattern
https://learn.microsoft.com/en-us/azure/architecture/patterns/scheduler-agent-supervisor

## Combination of different Queuing systems 
Since the Azure function that uses a service bus trigger it will poll the service bus after a certain period. This will cost compute for not reason if the service bus doesn't have messages coming in. So you could use event grid to only initiate reading when there is a message to process.

![](Images/Pasted%20image%2020251231145210.png)
https://learn.microsoft.com/en-us/azure/architecture/guide/technology-choices/messaging#crossover-scenarios