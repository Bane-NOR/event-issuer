
## How to consume events from Event Issuer

Use the subscriptions endpoint to subscribe to an Event.

> POST https://api.banenor.no/event-issuer/v1alpha/subscriptions

Example Payload

```json
{
    "applicationId": "application1",
    "event": "operational.event.train-arrived-at-station.v1",
    "url": "https://test.no/test"
}
```

| Parameter     | Description                                           |
|---------------|-------------------------------------------------------|
| applicationId | Identificator of Application that subscribes to event |
| event         | Which Event type to subscribe to                      |
| url           | The endpoint to return events to                      |

If subscription is created successfully, Event Issuer will start sending events to the url given in the subscription.
The events are sent as `application/json` and contains the key and value of the Event.

```json
{
    "key": "object",
    "value": "object"
}
```

The deserialization and further consumption should be handled by the subscriber.
