!!! warning under construction
    The following section is under construction

# Subscriptions

Subscriptions is the main mechanism for getting realtime events from the Bane NOR event backbone. The subscription is an reference to an application that wants events to be sendt to an webhook endpoint. The subscriber can configure the authentication towards the endpoint in addition to an API key if that is needed.

## Webhook endpoint

The endpoint can receive the event payload with additional metadata by using the [CloudEvents](cloudevents.md) HTTP binding.

Cloud events are sent by using the [HTTP Protocol Binding](https://github.com/cloudevents/spec/blob/v1.0.2/cloudevents/bindings/http-protocol-binding.md). This means that the CloudEvents are part of the HTTP headers.

!!! info
    Bane NOT is workign on stanarising on the cloud event specification wich means that some event types might be missing the cloud event headers.

## Authenticating subscriptions

Some users require authentication and authorization to be able to communicate with their APIs. For this purpose different types can be configured for the subscription. The following are supported:

- No authentication
- [API Key](https://swagger.io/docs/specification/authentication/api-keys/)
- [Basicauth](https://swagger.io/docs/specification/authentication/basic-authentication/)
- [OAuth 2.0](https://swagger.io/docs/specification/authentication/oauth2/)

The idea is that the end users or services can configure the needed information to authenticate towards the webhook endpoint and update the configuration as needed.

The API Key can be configured with any of the other options in case it is needed as part of your API management setup.

### API Key

API Key is something that usualy are created when subscribint to certain API or products that can be used both as an token for an API Management system to check if the request should be handeled and in monitoring siuations to check the number of requests are used with example rate limiting.

Event Issuer supports the use of an API Key configuration wich can be configured with one of the other autentication mechanisms if both are needed.

### Basic Auth

Basic authentication with an username and password is supported as an simple mechanism for getting access to an webhook.

#### Example

```json
{
    "applicationId": "application1",
    "eventName": "cloud.operational.train-arrived-at-station.v1",
    "url": "https://test.no/test",
    "apiKey": {
        "header": "Ocp-Apim-Subscription-Key",
        "key": "jnfdi923r8fnaszy12orf98032nrcn7u982"
    },
    "authentication": {
        "type": "basic",
        "username": "systemx",
        "password": "use-a-secure-password"
    }
}
```

### Identity Providers

An [identity provider (IdP)](https://en.wikipedia.org/wiki/Identity_provider) is a system that creates, stores, and manages digital identities. The IdP can either directly authenticate the user or can provide authentication services to third-party service providers (apps, websites, or other digital services).

The following IdPs are supported by to fetch OAuth2.0 tokens:

- [Maskinporten](https://www.digdir.no/felleslosninger/maskinporten/869)
- [Entra ID](https://learn.microsoft.com/en-us/entra/fundamentals/whatis)

The following diagram shows the system-context for the communication with and IdP.

![system-context](../img/subscriptions/identity-provider.drawio.svg)

#### Maskinporten

[Maskinporten](https://www.digdir.no/felleslosninger/maskinporten/869) is a Norwegian solution to add authorisations between companies that needs to be able to share data between systems or in other words machine-to-machine.

##### Example

```json
{
    "applicationId": "application1",
    "eventName": "cloud.operational.train-arrived-at-station.v1",
    "url": "https://test.no/test",
    "apiKey": {
        "header": "Ocp-Apim-Subscription-Key",
        "key": "jnfdi923r8fnaszy12orf98032nrcn7u982"
    },
    "authentication": {
        "type": "maskinporten",
        "scopes": [ "company:apix:write" ]
}
```

!!! info
    To be able to use Maskinporten the integration team at Bane NOR needs to be contacted to link the Maskinporten integration to be abele to use the correct scope from the API vendor.

For companies that uses an [token exchange](https://datatracker.ietf.org/doc/html/rfc8693) where the maskinporten token needs to be exchagned with an company specific token, contact the integration team with the speicif needs. This will then be added to the backlog and implemented as an tenant specific solution for those needs.

#### Entra ID

[Entra ID](https://learn.microsoft.com/en-us/entra/fundamentals/whatis) is the standard OAuth authentication mechanism used in Azure. To be able to use this with Event Issuer subscriptions, we will need to do a server-to-server interaction that runs in the background, without immediate interaction with a user. This is done through an OAuth client-credential flow that grants permissions directly to the application itself by an administrator.

Entra Id also uses the [JWT Grant mechanisms](https://datatracker.ietf.org/doc/html/rfc7523) to obtain the `access_token` for requests. These are the configuration options for Entra.
``` json
// With client secret
{
    "AuthUrl": "https://login.microsoftonline.com/",
    "TenantId": "[Enter here the tenantID or domain name for your Azure AD tenant]",
    "ClientId": "[Enter here the ClientId for your application]",
    "ClientSecret": "string",
    "scope": "string",
    "grant_type": "client_credentials"
}
```

``` json
// With certificate or federated credential
{
    "AuthUrl": "https://login.microsoftonline.com/",
    "TenantId": "[Enter here the tenantID or domain name for your Azure AD tenant]",
    "ClientId": "[Enter here the ClientId for your application]",
    "client_assertion_type": "The value must be set to urn:ietf:params:oauth:client-assertion-type:jwt-bearer.",
    "client_assertion": "JSON web token needed to sign with the certificate",
    "scope": "string",
    "grant_type": "client_credentials"
}
```

##### Example

```json
{
    "applicationId": "application1",
    "eventName": "cloud.operational.train-arrived-at-station.v1",
    "url": "https://test.no/test",
    "apiKey": {
        "header": "Ocp-Apim-Subscription-Key",
        "key": "jnfdi923r8fnaszy12orf98032nrcn7u982"
    },
    "authentication": {
        "type": "entraid",
        "url": "https://login.microsoftonline.com/{tenantId}/oauth2/v2.0/token",
        "clientId": "d0a3da30-8936-4800-9c23-37c1b86d8a63",
        "clientSecret": "7hzQ3bPSNThb7Cgem+a+w2RqLMKr*LqCSALYco-zQyi4ueUnVo",
        "scopes": [ "apix:write" ]
}
```

##### Client credentials flow

This diagram describes how authentication works between background services. For Event-Issuer the flow would look like this.

**This flow assumes that an admin has created an app registration for the subscription and given it the correct permissions to the Web API in questions.**

``` mermaid
sequenceDiagram
    participant Event Issuer Subscription
    participant EntraId
    participant Web API

    Event Issuer Subscription->>EntraId: Request token
    EntraId-->>Event Issuer Subscription: Returns token
    loop Until the consumer has caught up to the offset
    Event Issuer Subscription->>Web API: Posts data to API with token in Authorization header
    end
    Web API->>Web API: Validates token
    alt Success
    Web API-->>Event Issuer Subscription: 200 ok
    else Failed
    Web API -->>Event Issuer Subscription: 401, 403, 404, 501, etc.
    end
```

##### Access control

Microsoft provides two options to grant access to applications: **access control lists** and **application permissions**.

###### Access control lists

Access control lists enforce autorization based on a list of applicaiton IDs that it knows and grants a specific level of access to. When the relevant resource receives a token, it decodes it and checks it against the list of authorized clients.

###### Application permissions

For data owned by organizations, Microsoft recommends using application permissions. To use application roles with your own API, you need to expose the app roles in the API's app registration, then configure the required roles in your client's (the subscription) app registration. The user who creates a subscription will also have to create an app registration in their organization that they provide the neccessary permissions.

## Create Subscription

To create an subscription the following command can be used to create an subscription that needs to use an api key and basic authentication.

```curl
curl -H "Ocp-Apim-Subscription-Key: ApiKey" https://<bane-nor-api-endpoint>/event-issuer/v1alpha/{tenantId}/subscriptions -d '{"applicationId": "my-applicaiotn", "event": "event-name", "url": "https://my-endpoint.com/events}, "apiKey": { "header": "Ocp-Apim-Subscription-Key", "key": "api-key" }, "authentication": { "username": "user1", "password": "my-secure-password" }'
```

If only an api key is needed don't set the `authentication` and and only the `apiKey` section.
