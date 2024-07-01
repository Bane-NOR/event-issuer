# Error reporting
Currently Event Issuer users are not able to see why their subscriptions are failing. With the Event Issuer Error Report feature, we want to make it easier for users to see their subscription's errors.


# Design

## Get Errors url
Errors will be presented together with a subscription, so getting errors will be the same URLs as getting subscriptions

### Specific subscription
**GET** http://{env}.api.apps.banenor.no/event-issuer/v1alpha/{tenantId}/subscriptions/{subscriptionId}

### All subscriptions within a tenant
**GET** http://{env}.api.apps.banenor.no/event-issuer/v1alpha/{tenantId}/subscriptions

## Errors in Subscription
Instead of creating a separate table for SubscriptionErrors, we could add it as a seperate entity to the Subscription table. Adhering to the Azure Storage table [compound key pattern](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-patterns#compound-key-pattern:~:text=heterogeneous%20entity%20types-,Compound%20key%20pattern,-Use%20compound%20RowKey). 

In this table we have added the Error entity to the Subscription table. Note how the partition key is the same on both entities while the RowKeys both contain the subscription id, but the Error entity has some extra parameters. This allows us to create a one-to-many releation ship between subscriptions and errors and allows us to search errors. 

![table-entities](../img/subscriptions/event-issuer-error-reporting-diagram.drawio.png)

### EntityType
In both these entities, we have added the field **EntityType** this field will allow us to get both entities in a single query as shown by Microsoft in their [documentation](https://learn.microsoft.com/en-us/azure/storage/tables/table-storage-design-patterns#working-with-heterogeneous-entity-types:~:text=Retrieving%20heterogeneous%20entity%20types).

```csharp
Pageable<TableEntity> entities = tableClient.Query<TableEntity>(x =>
    x.PartitionKey ==  subscriptionId)

foreach (var entity in entities)
{
    if (entity.GetString("EntityType") == nameof(SubscriptionAzureTable))
    {
        var subscription = JsonSerializer.Deserialize<SubscriptionAzureTable>(JsonSerializer.Serialize(error));
        // use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
    }

    if (entity.GetString("EntityType") == nameof(SubscriptionErrorTable))
    {
        var error = JsonSerializer.Deserialize<SubscriptionErrorTable>(JsonSerializer.Serialize(error));
        // use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
    }
} 
```

### Event Issuer storing errors
When storing errors, the errors will source of the errors will have to implement and publish a mediatoR request of type CreateSubscriptionErrorEvent containing the error.

::: mermaid
sequenceDiagram
    participant Error Source
    participant MediatR
    participant CreateSubscriptionErrorCommandHandler
    participant UnitOfWork
    participant Storage Account

    Error Source->>MediatR: Send CreateSubscriptionErrorEvent
    MediatR->>CreateSubscriptionErrorCommandHandler: Handle CreateSubscriptionErrorEvent
    CreateSubscriptionErrorCommandHandler ->> UnitOfWork: Call unit of work to submit transaction
    UnitOfWork ->> Storage Account: Submit transaction 
    Storage Account -->> UnitOfWork: Return successful operation
    UnitOfWork -->> CreateSubscriptionErrorCommandHandler: Return successful operation
    CreateSubscriptionErrorCommandHandler -->> MediatR: Return successful operation
    MediatR -->> Error Source: Return successful operation
    alt
    Storage Account -->> UnitOfWork: Return failed operation
    UnitOfWork -->> CreateSubscriptionErrorCommandHandler: Return failed operation
    CreateSubscriptionErrorCommandHandler -->> MediatR: Return failed operation
    MediatR -->> Error Source: Return failed operation
    end
:::


## User process querying for subscription errors

To retrieve a subscription with the errors, we need to modify the SubscriptionResult record to include a list of the ProblemDetails as well. 

::: mermaid
sequenceDiagram
    participant External User
    participant API Management
    participant Event Issuer API
    participant SubscriptionQuery
    participant Storage Account

    External User->>API Management: Request subscription errors
    API Management->>Event Issuer API: Call Event Issuer
    Event Issuer API->>SubscriptionQuery: Send new GetErrorsForSubscription request
    SubscriptionQuery ->> Storage Account: Retrieve errors from storage account
    Storage Account -->> SubscriptionQuery: Return result
    SubscriptionQuery -->> Event Issuer API: Return result
    Event Issuer API -->> API Management: Return result
    API Management -->> External User: Return errors to end user
:::