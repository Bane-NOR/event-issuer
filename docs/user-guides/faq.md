[TOC]

Click on the questions to reveal the answer. If you cant find the answer you are looking for, please let us know by contacting [integrasjonsteamet@banenor.no](mailto:integrasjonsteamet@banenor.no).

## Q: I regenerated my API keys, but i no longer have access
When you regenerate your API Keys in the API portal, you get new API-keys that are usable for the Event-issuer API found in APIM. However, the new keys are now no longer linked to the policy in Event Issuers Authorization Service, meaning that it will stop you when you try to complete the same actions as before such as CreateSubscription or ProduceEvent. To fix this issue, please get in touch with a member of the integration team.

## Q: I am not allowed to produce an event/topic with event issuer, even though my confluent user has access
When you try to produce data for an event or topic that you have a confluent user for, it will usually allow you to complete the requested action. However, Event-Issuer has an extra authorization layer that also validates that your API-Subscription Keys are allowed to produce data for the specified event. To also get this access, please contact a member of the integration team. 