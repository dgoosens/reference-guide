# Consumer

When [Message Endpoints](message-endpoint/) are connected to channels and instantiated, they produce one of the following instances:

* `Event Driven Consumer`
* `Polling Consumer`

## Event Driven Consumer 

Event Driven Consumers are automatically called when the message arrives on the channel.   
They are connected in synchronous manner, which leads to [Endpoint](message-endpoint/) being called just after [Message](message.md) arrives on the [channel](message-channel.md).

## Polling Consumer

Polling consumers let actively poll for [Messages](message.md) rather than process messages in an event-driven manner.   
The Polling Consumer is created, when Endpoint is connected to `Pollable Channel.`   
Polling Consumer is running in separate process. 

{% hint style="info" %}
You will see how easily they are connected in [Asynchronous section](../scheduling.md).
{% endhint %}

## Consumer Abstraction

The `consumer abstraction` \(Polling/Event Driven\), which is automatically created based on connected channel takes responsibility from the developer, to create and maintain consumers manually.   
This process becomes dynamic and automatic and with low cost create possibility to pretty easily move from asynchronous code to synchronous and vice versa.   
Testing asynchronous code can be really cumbersome, but thanks to the abstraction, we can replace asynchronous channel with synchronous in tests. Which will lead to situation where endpoint  in tests will be directly called, because `Event Driven Consumer` will be created.  


{% hint style="info" %}
The main idea behind `Ecotone` is to handle integration logic.   
Developer should not need to bother with creating consumers, testing asynchronous code.   
The best code is the code, which is not aware of being asynchronous code.
{% endhint %}

