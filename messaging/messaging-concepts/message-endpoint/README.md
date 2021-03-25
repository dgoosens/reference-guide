---
description: Message Endpoint PHP
---

# Message Endpoint

![](../../../.gitbook/assets/endpoint-1.jpg)

Message Endoints are consumers and producers of messages. Consumer are not necessary asynchronous, as you may build synchronous flow, compound of multiple endpoints.   
You will not have to implement them directly, as you should not even have to build messages and invoke or receive message directly from the [Message channel](../message-channel.md). Instead you will be able to focus on your specific domain model with with an implementation based on plain PHP objects. By providing declarative configuration, you can "connect” your domain-specific code to the messaging infrastructure provided by _Ecotone_. 

{% hint style="info" %}
`Command Handlers, Event Handlers, Query Handlers` are also Endpoints.
{% endhint %}

