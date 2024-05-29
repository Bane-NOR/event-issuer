# Error reporting
Currently Event Issuer users are not able to see why their subscriptions are failing. With the Event Issuer Error Report feature, we want to make it easier for users to see their subscription errors.


# Design

## Get Errors url

**GET** http://{env}.api.apps.banenor.no/event-issuer/v1alpha/{tenantId}/subscriptions/{subscriptionId}/errors

## SubscriptionError table
Error table: SubscriptionErrors. With dummy data

- **PartitionKey**: Required key for Azure Storage Table
- **RowKey**: Required key for Azure Storage Table
- **Id**: Unique identifier for the specific error messages.
- **SubscriptionId**: The subscription in which the error messages belongs to.
- **Timestamp**: The time the error was registered.
- **TraceId**: The identifier to the trace of the workload.
- **StatusCode**: The status code for the error message.
- **ErrorMessage**: The error that occurred in Event Issuer.
- **UserErrorMessage**: The error message exposed to the user.

| **PartitionKey** | **RowKey** |**Id**   | **SubscriptionId**                     | **Timestamp**                 |**TraceId**                             |**StatusCode**|**ErrorMessage**   | **UserErrorMessage** |
|-|-|------|----------------------------------------|-------------------------------|----------------------------------------|--------------|--------------------|-|
|a1f04635-87a5-4fa7-8954-95fc6a5073e2 | 2a20cdd2-ff4b-4a05-bb2f-7f6fafb059a7|   e7e2a789-5fed-44d8-9fdd-0b2998a7d231   |bf40122b-b74a-4720-bb81-5c3322d68921  |2024-05-13T09:49:45.0245460Z   |    0e5b736b-1d3d-4a29-b78c-1bd3b5ac9bce. |      500     |[thrd:app]: rdkafka#consumer-1: GroupCoordinator: b0.staging.eda.banenor.no:443: SSL handshake failed: Disconnected: connecting to a PLAINTEXT broker listener? (after 34ms in state SSL_HANDSHAKE)| Failed forwarding message to endpoint: {endpoint}. Event Issuer internal error |


## Event Issuer storing errors

::: mermaid
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
:::

## User process querying for subscription errors

::: mermaid
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
:::