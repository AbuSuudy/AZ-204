# Event Grid
Event grid is push notification system that can use HTTP and MQTT protocols. Removes the need for constant polling. Event are sent from the source to the event grid. The event grid is responsible to deliver the messages to the event handlers.

![](Images/Pasted%20image%2020251218203435.png)

- *Event* - What happened 
- *Event Source* - Where the event took place
- *Topic* - Where publisher send message to on the event grid. A collection of related events.
- *Event Subscription* -  Defines how messages from the topic is delivered to the event hander: destination, retries and dead letter queues.
- *Event Handler* - The app reacting to the event 

![](Images/Pasted%20image%2020251220003600.png)

## Event
Example of event schema sent to event grid. `Subject`, `eventType`, `eventTime` and `Id` are mandatory. 

```json
[
  {
    "topic": "/subscriptions/providers/Microsoft.Storage/storageAccounts/mystorage",
    "subject": "/blobServices/default/containers/images/blobs/vacation.jpg",
    "eventType": "Microsoft.Storage.BlobCreated",
    "eventTime": "2025-12-18T21:41:38.123456Z",
    "id": "831e1144-7777-4444-8888-32038531135d",
    "data": {
      "api": "PutBlob",
      "clientRequestId": "6d79dbfb-0e37-4144-904c-7c0c32688000",
      "requestId": "fb37e000-0001-001f-6500-111111110000",
      "eTag": "0x8D4BCC2E4835300",
      "contentType": "image/jpeg",
      "contentLength": 45812,
      "blobType": "BlockBlob",
      "url": "https://mystorage.blob.core.windows.net/images/vacation.jpg",
      "sequencer": "000000000000000000000000000001D300000000000001a1",
      "storageDiagnostics": {
        "batchId": "68147a00-0001-0011-0333-555555555555"
      }
    },
    "dataVersion": "2",
    "metadataVersion": "1"
  }
]
```

## MQTT Messaging
MQTT allows you to use publish-subscribe messaging model. 

![](Images/Pasted%20image%2020251218205349.png)

## HTTP Messaging 
Event Grid supports push and pull event delivery by using HTTP. With _push delivery_, you define a destination in an event subscription, to which Event Grid sends events. With _pull delivery_, subscriber applications connect to Event Grid to consume events.

> [!NOTE] 
> HTTP has both push and pull methods so can be more flexiable than MQTT

![](Images/Pasted%20image%2020251218210240.png)

## Delivery
*Push* uses exponential back off and if the messages isn't delivered in 24 hours it's deleted or could be configured to add to a storage account that can act as dead letter. If you use *pull* delivery, your application has full control over event consumption.

## Push vs Pull
### Pull
- You need full control over when to receive events . Down stream service may be over whelmed.
- You want to use private links when you receive events, which is possible only with the pull delivery, not the push delivery.

### Push
- You want to avoid constant polling to determine that a system state change occurred.

## Event Grid vs Blob Trigger
For a function app there are two binding related to blob storage update. One used event grid and other uses blob trigger.

```c#
 [BlobTrigger("test-samples-trigger/{name}")]
 [EventGridTrigger] MyEventType input
```

With the blob trigger the function app will only trigger once the function wakes up and polls the blob storage account which can take up to 10 min.

Where event grid it get a push notification directly from source and causes the function to wake up and handle the event as it comes in. If low latency is required used event grid. Unless you use plan that never scale to 0 and always has warmed up function ready.

> [!NOTE] 
>  Blobs are scanned in groups of 10,000 at a time with a continuation token used between intervals. If your function app is on the Consumption plan, there can be up to a 10-minute delay in processing new blobs if a function app has gone idle. The larger the storage account the longer it will take.

## Native Event Support instead of using Event Grid
Some resources have built in integration for event e.g. trigger function app if blob storage changes happen.  Azure uses system topic behind the scenes. 

If you create event grid topic you can have multiple apps subscribe to that topic and respond to the event from one place. Also could be used to notify another system once function app has finished processing that blob file.

## Custom Topic
An Event Grid topic provides an endpoint where the source sends events. Custom event allow applications that are not native to azure to send event. An example would be an application running on prem that will publish events. The event JSON payload will need to fit in certain schema. 

*Azure Event Grid Schema*

```json
[
  {
    "id": "12345",
    "eventType": "OrderCreated",
    "subject": "orders/customer-99",
    "eventTime": "2025-12-20T10:00:00Z",
    "data": {
      "orderId": "abc-123",
      "amount": 49.99,
      "currency": "USD"
    },
    "dataVersion": "1.0"
  }
]
```

```c#
using Azure.Messaging.EventGrid;

string eventGridTopic = "";
string key = "";

EventGridPublisherClient client = new EventGridPublisherClient(
	new Uri(eventGridTopic),
	new Azure.AzureKeyCredential(key)
 );

EventGridEvent egEvent =
	new EventGridEvent(
		"ExampleEventSubject",
		"Example.EventType",
		"1.0",
	  new { Data = "Payload", id = Guid.NewGuid() });

await client.SendEventAsync(egEvent);
```

*Cloud Events schema* which is cloud agnostic way to represent events. You could prevent breaking changes by adding versioning of your types if data payload changes. 

```json 
{
  "specversion": "1.0",
  "type": "com.contoso.order.created.v1",
  "source": "/onprem/ordersystem",
  "id": "f4b2c2e1-3c89-4f1a-8c42-0f6a2e5c91d4",
  "time": "2025-03-01T10:15:30Z",
  "datacontenttype": "application/json",
  "subject": "order/12345",
  "data": {
    "orderId": "12345",
    "customerId": "C001",
    "total": 250.75
  }
}
```

If you want to create event grid topic that accepts Cloud events this is only done on the CLI

```bash
az eventgrid topic create --name demotopic -l uksouth -g Test --input-schema cloudeventschemav1_0
```

Minimal example of post cloud event to event grid topic. 

```c#
using Azure.Messaging.EventGrid;
using CloudNative.CloudEvents;
using Microsoft.Azure.Messaging.EventGrid.CloudNativeCloudEvents;

string eventGridTopic = "";
string key = "";

EventGridPublisherClient client = new EventGridPublisherClient(
	new Uri(eventGridTopic),
	new Azure.AzureKeyCredential(key)
);

var cloudEvent = new CloudEvent
{
	Id = Guid.NewGuid().ToString(),
	Type = "record",
	Source = new Uri("http://www.contoso.com"),
	Data = "data"
};

await client.SendCloudNativeCloudEventAsync(cloudEvent);
```

Event grid Topic with storage queue subscription for events to be saved.

![](Images/Pasted%20image%2020251220231756.png)

The storage queue with event

![](Images/Pasted%20image%2020251220231857.png)