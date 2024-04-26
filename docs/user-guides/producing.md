!!! warning under construction
    The following section is under construction

# Producing

Event Issuer can be used to produce new events to Bane NOR. This is not openly available and an aggreement with Bane NOR is needed to be able to produce data. The correct access rights for producing will given on an aggreement basis.

Bane NOR uses the [Cloud Event](https://cloudevents.io/) specification for event that needs to be produced and support both single event producers and the possibility to send an batch of events.

Cloud Events has created SDKs for different languages that can be found on the main page under the `SDKs`in the menu.

The Event Issuer has two API endpoints for this:

- {tenantId}/produce
- {tenantId}/produce/batch

## Cloud Events

Cloud Events supports different ways of sending mainly, structured, binary and batch. At the moment only JSON is supported to be used, but other content types can be added later.

### Binary

In binary mode the `cloud event` headers are sent as part of the HTTP header values by using the `ce-` prefix. For more information about this see the [binary mode](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/http-protocol-binding.md#31-binary-content-mode) documented in the specification.

```http
POST event-issuer/v1/{tenantId}/produce HTTP/1.1
Host: api.banenor.com
ce-specversion: 1.0
ce-type: cloud.domain.sub-domain.event.v1
ce-time: 2018-04-05T03:56:24Z
ce-id: 1234-1234-1234
ce-source: /mycontext/subcontext
    .... further attributes ...
Content-Type: application/json; charset=utf-8
Content-Length: nnnn

{
    ... application data ...
}
```

### Structured

With structured modes the `cloud event` headers are sent as part of the HTTP payload/body data. For more information about this see the [structured mode](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/http-protocol-binding.md#32-structured-content-mode) documented in the specification.

```http

POST event-issuer/v1/{tenantId}/produce HTTP/1.1
Host: api.banenor.com
Content-Type: application/cloudevents+json; charset=utf-8
Content-Length: nnnn

{
    "specversion" : "1.0",
    "type" : "cloud.domain.sub-domain.event.v1",

    ... further attributes omitted ...

    "data" : {
        ... application data ...
    }
}
```

### Batch

With batch mode and list of cloud events can be sendt in the same HTTP request that needs to be produced. This a list of event in an structured mode.

```http
POST event-issuer/v1/{tenantId}/produce/batch HTTP/1.1
Host: api.banenor.com
Content-Type: application/cloudevents-batch+json; charset=utf-8
Content-Length: nnnn

[
    {
        "specversion" : "1.0",
        "type" : "cloud.domain.sub-domain.event.v1",

        ... further attributes omitted ...

        "data" : {
            ... application data ...
        }
    },
    {
        "specversion" : "1.0",
        "type" : "cloud.domain.sub-domain.event.v2",

        ... further attributes omitted ...

        "data" : {
            ... application data ...
        }
    }
]
```
