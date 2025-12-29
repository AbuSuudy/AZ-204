# Choose between Azure messaging services

## Messages
There two types of messages 
### Events
An event is a lightweight notification of a condition or a state change. The publisher of the event has no expectation about how the event is handled. The event data has information about what happened but doesn't have the data that triggered the event. For example, an event notifies consumers that a file was created.
### Commands
A message is raw data produced by a service to be consumed or stored elsewhere. The message contains the data that triggered the message pipeline. The publisher of the message has an expectation about how the consumer handles the message. The producer may wait for response from the consumer once it's done processing. Only after receiving the response does the producer send a new message to start the next operation in the sequence.

![](Images/Pasted%20image%2020251229171531.png)
## Comparison Between Services

| Service     | Direction | Purpose                                                                     | Type                                 | When to use                                |
| ----------- | --------- | --------------------------------------------------------------------------- | ------------------------------------ | ------------------------------------------ |
| Event Grid  | Push      | Reactive programming                                                        | Event distribution (discrete events) | React to status changes                    |
| Event Hubs  | Pull      | Big data pipeline                                                           | Event streaming (series)             | Telemetry and distributed data streaming   |
| Service Bus | Pull      | High-value enterprise messaging that can't be dropped or accept duplicates. | Message                              | Order processing and financial transaction |

## Use Together
