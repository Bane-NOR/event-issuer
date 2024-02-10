# Subscriptions

Subscriptions is the main mechanism for getting realtime events from the Bane NOR event backbone. The subscription is an reference to an application that wants events to be sendt to an webhook endpoint. The subscriber can configure the authentication towards the endpoint in addition to an API key if that is needed.

## Create Subscription

To create an subscription the following command can be used to create an subscription that needs to use an api key and basic authentication.

```curl
curl -H "Ocp-Apim-Subscription-Key: ApiKey" https://<bane-nor-api-endpoint>/event-issuer/v1alpha/{tenantId}/subscriptions -d '{"applicationId": "my-applicaiotn", "event": "event-name", "url": "https://my-endpoint.com/events}, "apiKey": { "header": "Ocp-Apim-Subscription-Key", "key": "api-key" }, "authentication": { "username": "user1", "password": "my-secure-password" }'
```

If only an api key is needed don't set the `authentication` and and only the `apiKey` section.

## Webhook endpoint

The endpoint will receive the event payload with additional metadata by using the [CloudEvents](cloudevents.md) HTTP binding.

The following header values will be found as part of the request:

```
ce-specversion: 1.0
ce-type: no.banenor.event-type
ce-id: <unique-id>
ce-source: <source identification of the event>
ce-time: 2024-01-05T03:56:24Z
ce-datacontenttype: application/json
```

!!! info
    As event issuer is under development this can change and/or some are missing. Event Issuer is also dependent on publishers using cloud events as not all are using them at the moment.
