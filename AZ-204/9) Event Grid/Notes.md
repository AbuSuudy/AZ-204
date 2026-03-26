# Event Grid
Event grid is push notification system that can use HTTP and MQTT protocols. Removes the need for constant polling. Event are sent from the source to the event grid. The event grid is responsible to deliver the messages to the event handlers.

![](Images/Pasted%20image%2020251218203435.png)

- *Event* - What happened 
- *Event Source* - Where the event took place
- *Topic* - Where publisher send message to on the event grid. A collection of related events.
- *Event Subscription* -  Defines how messages from the topic is delivered to the event hander: destination, retries and dead letter queues.
- *Event Handler* - The app reacting to the event 

![](Images/Pasted%20image%2020251220003600.png)

Types of endpoint Event grid notifies:
![](Images/Pasted%20image%2020260326185049.png)
## System Topic vs Custom Topic
*System Topic* in Event Grid represents one or more events published by Azure services.

*Custom Topic* provides an endpoint publish event from 3rd party application e.g. software running in azure vm/ application running on prem or an Azure service that isn't integrated with Event Grid

## Event Grid Trigger vs Blob Trigger
For a function app there are two binding related to blob storage update. One used event grid and other uses blob trigger.

```c#
 [BlobTrigger("test-samples-trigger/{name}")]
 [EventGridTrigger] MyEventType input
```

> [!NOTE] 
>  When using `BlobTrigger` blobs are scanned in groups of 10,000 at a time with a continuation token used between intervals. If your function app is on the Consumption plan, there can be up to a 10-minute delay in processing new blobs if a function app has gone idle. The larger the storage account the longer it will take.

When you use event grid the event get pushed to the function app and causes it to wake up and process the data as it comes it. If low latency is critical you should be using event grid.

## MQTT Messaging
MQTT allows you to use publish-subscribe messaging model. 

![](Images/Pasted%20image%2020251218205349.png)

## HTTP Messaging 
Event Grid supports push and pull event delivery by using HTTP. With _push delivery_, you define a destination in an event subscription, to which Event Grid sends events. With _pull delivery_, subscriber applications connect to Event Grid to consume events.

> [!NOTE] 
> HTTP has both push and pull methods so can be more flexiable than MQTT

![](Images/Pasted%20image%2020251218210240.png)

## Delivery
*Push* uses exponential back off and if the messages isn't delivered in 24 hours it's deleted or could be configured to add to a storage account that can act as dead letter. If you use *pull* delivery, your application has full control over the rate of consumption.

## Push vs Pull
### Pull
- You need full control over when to receive events . Down stream service may be over whelmed.
- You want to use private links when you receive events, which is possible only with the pull delivery, not the push delivery.

### Push
- You want to avoid constant polling to determine that a system state change occurred.

## Native Event Support Resources
Some resources have built in integration for event e.g. trigger function app if blob storage changes happen. This done on the event tab. It uses system event topic behind the scenes. 

If you create a event grid topic resource you can have multiple apps subscribe to that topic and respond to the event in one place. 

![](Images/Pasted%20image%2020251222213537.png)

## Posting Events to Custom Topic
*Azure Event Grid Schema* - `Subject`, `eventType`, `eventTime` , `Data` are the only required fields.

```json
{
	"topic": string,
	"subject": string,
	"id": string,
	"eventType": string,
	"eventTime": string,
	"data":{
	  object-unique-to-each-publisher
	},
	"dataVersion": string,
	"metadataVersion": string
}
```

```c#
using Azure.Messaging.EventGrid;

public static async Task PostEventGridSchema()
{
	string eventGridTopic = "";
	string key = "";
	
	EventGridPublisherClient client = new EventGridPublisherClient(
		new Uri(eventGridTopic),
		new Azure.AzureKeyCredential(key)
	 );
	 
	EventGridEvent eventObj = new EventGridEvent(
		subject: "ExampleEventSubject",
		eventType: "Example.EventType",
		dataVersion: "1.0",
		data: new { Data = "Payload", id = Guid.NewGuid() }
	 );
	 
	await client.SendEventAsync(eventObj);
}
```

*Cloud Events schema* which is cloud agnostic way to represent events. You could prevent breaking changes by adding versioning of your types if data payload changes. 

Required fields: `id`,`Souce`, `specversion` and `type`

```json 
{
    "specversion": string,
    "type": string,  
    "source": string,
    "id": string,
    "time": string,
    "subject": string,    
	"data":{
	  object-unique-to-each-publisher
	}
}
```

You will need to use CLI to  create event grid topic that accepts Cloud Event JSON schema 

```bash
az eventgrid topic create --name demotopic -l uksouth -g Test --input-schema cloudeventschemav1_0
```

Minimal example of post cloud event to event grid topic. 

```c#
using Azure.Messaging.EventGrid;
using CloudNative.CloudEvents;
using Microsoft.Azure.Messaging.EventGrid.CloudNativeCloudEvents;

public static async Task PostCloudEventSchema()
{
	string eventGridTopic = "";
	string key = "";
	
	EventGridPublisherClient client = new EventGridPublisherClient(
		new Uri(eventGridTopic),
		new Azure.AzureKeyCredential(key)
	);
	
	//Required: Id, source, specVersion, type
	//SpecVersion is read only and default to v1 since it's
	//only one version of the Cloud Event Spec.
	var cloudEvent = new CloudEvent
	{
		Id = Guid.NewGuid().ToString(),
		Source = new Uri("http://www.contoso.com"),
		Type = "Event.v1"
	};
	
	await client.SendCloudNativeCloudEventAsync(cloudEvent);
}
```

Event grid Topic with storage queue subscription for events to be saved.

![](Images/Pasted%20image%2020251220231756.png)

The storage queue with event

![](Images/Pasted%20image%2020251220231857.png)