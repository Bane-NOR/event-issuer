# Cloudevents

The [cloudevents specification](https://cloudevents.io/) is used by Event Issuer both for subscribed events and produced events. The difference is how Event Issuer is using the Cloud Events for events being subscribed to and when producing.

## Subscription

Subscribed events are receiving the cloud events by using the [HTTP Protocol Binding](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/http-protocol-binding.md). This means that the CloudEvents are part of the HTTP headers.

## Producers

Producers needs to transmit the CloudEvent metadata by using the [JSON Event Format](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/formats/json-format.md). The produce endpoints expect a content type of `application/cloudevents+json` as specified in section `3. Envelope` of the specification.
