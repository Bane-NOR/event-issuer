# Authenticating subscriptions
Some users require authentication and authorization to be able to communicate with their APIs. For this we will support Maskinporten and Entra Id from Microsoft.
## Entra Id
Entra Id is the standard OAuth authentication mechanism used in Azure. To be able to use this with Event Issuer subscriptions, we will need to do a server-to-server interaction that runs in the background, without immediate interaction with a user. This is done through an OAuth client-credential flow that grants permissions directly to the application itself by an administrator.

### Client credentials flow 
This diagram describes how authentication works between background services.

For Event-Issuer the flow would look like this
``` mermaid
sequenceDiagram
    participant Admin
    participant App registration
    participant Event Issuer Subscription
    participant EntraId
    participant Web API

    Admin->>App registration: Creates an app registration
    App registration-->>Admin: App registration is created
    Admin->>Event Issuer Subscription: Provides the app registration details when creating the subscription
    Event Issuer Subscription->>EntraId: Request token
    EntraId-->>Event Issuer Subscription: Returns token
    Event Issuer Subscription->>Web API: Posts data to API with token in Authorization header
    Web API->>Web API: Validates token
    alt Success
    Web API-->>Event Issuer Subscription: 200 ok
    else Failed
    Web API -->>Event Issuer Subscription: 401, 403, 404, 501, etc.
    end
```

### Access control
Microsoft provides two options to grant access to applications: **access control lists** and **application permissions**.
#### Access control lists
Access control lists enforce autorization based on a list of applicaiton IDs that it knows and grants a specific level of access to. When the relevant resource receives a token, it decodes it and checks it against the list of authorized clients.
#### Application permissions
For data owned by organizations, Microsoft recommends using application permissions. To use application roles with your own API, you need to expose the app roles in the API's app registration, then configure the required roles in your client's (the subscription) app registration. The user who creates a subscription will also have to create an app registration in their organization that they provide the neccessary permissions.