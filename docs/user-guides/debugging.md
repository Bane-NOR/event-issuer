# Debugging subscriptions
To make the event issuer more self-service, the integration team has set up multiple solutions to allow for as much debugging as possible on the client side before a member of the integration team has to get involved. The main methods recommended for debugging by the integration team are:

- Checking subscription status

## Debugging by checking subscription status
The Event Issuer has built-in Error Reporting linked to the GetSubscription methods. This means that if an error has occurred on your subscription, you can call the **GetSubscription** method which in case an error has occurred, will return an error report with useful debug information such as:

- A relevant Error Message
- A Trace ID for tracking the error in Grafana
- A HTTP status code 
- A timestamp for when the error occurred

An example of how this looks in the Bruno API Client is showcased below:
(Added when functionality is added)


