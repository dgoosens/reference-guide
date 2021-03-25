---
description: Enterprise Integration Patterns PHP
---

# Message Channel

![](../../.gitbook/assets/message-channel-connection.svg)

_`Message channel`_abstracts communication between components. It does allow for sending and receiving messages.  
A message channel may follow either point-to-point or publish-subscribe semantics.   
With a point-to-point channel, only one consumer can receive each message sent to the channel.   
Publish-subscribe channels, broadcast each message to all subscribers on the channel. 

_`Pollable channels`_ extends Message Channels with capability of buffering Messages within a queue. The advantage of buffering is that it allows for throttling the inbound messages and preventing of message loss. 

