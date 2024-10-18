# Event Issuer Terminology

This document contains concise explanations of important terminology found in the event issuer. For more specific information and use cases, read relevant documentation found on the event issuer Wiki.

| **Term**             | **Definition**                                                                                                         |
|----------------------|------------------------------------------------------------------------------------------------------------------------|
| **Consumer**         |  An entity either internal or external to Bane NOR which uses Event Issuer to consume data from the Bane NOR Event Backbone |
| **Producer**         |  An entity either internal or external to Bane NOR which uses Event Issuer to produce data to the Bane NOR Event Backbone |
| **Subscription**     |  A generic name used in Event Issuer to explain an active consumer within Event Issuer which continuously fetches data from the Bane NOR Event Backbone and forwards it to a specified webhook endpoint |
| **Tenant**           |  A Top-level entity representing organizations or more top-level structures. All principals, policies and subscriptions are linked to a tenant although some principals can also manage the tenant |
| **Principal**        |  A Principal is an object that represents a user, group, or service account. In the initial release mainly service accounts will be supported.|
| **Policy**           |  A Policy refers to authorization policies that determine which actions principals can take within the Event Issuer eco-system. Common policies would be principal policies for tenants, subscriptions, and events determining their possibility to create new subscriptions, list out subscriptions, delete active subscriptions, produce data, etc.|

## API Terminology

| **Term**             | **Definition**                                                                                                         |
|----------------------|------------------------------------------------------------------------------------------------------------------------|
| **Authentication**   | The process of verifying the identity of a user or system attempting to access an API.                                  |
| **Authorization**    | Determines what actions an authenticated user or system can perform.                                                   |
| **Endpoint**         | A specific URL where the API can receive requests, corresponding to a unique function or resource.                      |
| **Payload**          | The data transmitted in an API request or response in JSON.                          |
| **Status Codes**     | HTTP codes that indicate the result of an API request, such as 200 (success) or 404 (resource not found).               |
| **Webhook**          | A method for sending real-time data from the API to another system, triggered by an event.                             |
| **Access Token**     | A short-lived token used to access protected API resources, issued during authentication.                               |

