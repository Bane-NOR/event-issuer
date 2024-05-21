# Error reporting
Currently Event Issuer users are not able to see why their subscriptions are failing. With the Event Issuer Error Report feature, we want to make it easier for users to see their subscription errors.


# Design

## Get Errors url

**GET** http://{env}.api.apps.banenor.no/event-issuer/v1alpha//{tenantId}/subscriptions/{subscriptionId}/errors

## SubscriptionError table
Error table: subscriptionErrors. With dummy data

|        **PartitionKey**               |           **Row Key**                | **Timestamp**                 | **TenantId**                         | **SubscriptionId**                 | **ErrorMessage**   |
|---------------------------------------|--------------------------------------|-------------------------------|--------------------------------------|------------------------------------|--------------------|
|  012f9803-d5e3-470f-b484-ff076c1c94fa | 18a9e9b8-e8c1-423b-9f4a-4dd70187cfd6 |  2024-05-13T09:49:45.0245460Z | 4b7461d3-3501-42cd-812f-2837cd8aad33 |5b0448b1-e675-489f-8c86-a7cca7814db6| [thrd:app]: rdkafka#consumer-1: GroupCoordinator: b0.staging.eda.banenor.no:443: SSL handshake failed: Disconnected: connecting to a PLAINTEXT broker listener? (after 34ms in state SSL_HANDSHAKE)                 |

## Event Issuer storing errors

``` mermaid
sequenceDiagram
    participant Error Source
    participant MediatR
    participant SubscriptionErrorCommandHandler
    participant UnitOfWork
    participant Storage Account

    Error Source->>MediatR: Send SubscriptionErrorEvent
    MediatR->>SubscriptionErrorCommandHandler: Handle SubscriptionErrorEvents
    SubscriptionErrorCommandHandler ->> UnitOfWork: Call unit of work to submit transaction
    UnitOfWork ->> Storage Account: Submit transaction 
    Storage Account -->> UnitOfWork: Return successful operation
    UnitOfWork -->> SubscriptionErrorCommandHandler: Return successful operation
    SubscriptionErrorCommandHandler -->> MediatR: Return successful operation
    MediatR -->> Error Source: Return successful operation
    alt
    Storage Account -->> UnitOfWork: Return failed operation
    UnitOfWork -->> SubscriptionErrorCommandHandler: Return failed operation
    SubscriptionErrorCommandHandler -->> MediatR: Return failed operation
    MediatR -->> Error Source: Return failed operation
    end
```

## User process querying for subscription errors

``` mermaid
sequenceDiagram
    participant External User
    participant API Management
    participant Event Issuer API
    participant GetErrorsForSubscription
    participant Storage Account

    External User->>API Management: Request subscription errors
    API Management->>Event Issuer API: Call Event Issuer
    Event Issuer API->>GetErrorsForSubscription: Send new GetErrorsForSubscription request
    GetErrorsForSubscription ->> Storage Account: Retrieve errors from storage account
    Storage Account -->> GetErrorsForSubscription: Return result
    GetErrorsForSubscription -->> Event Issuer API: Return result
    Event Issuer API -->> API Management: Return result
    API Management -->> External User: Return errors to end user
```

## Questions

Should all error messages be allowed to be sent to the Data Store