---
description: Asynchronous PHP
---

# Asynchronous Handling

## Running Asynchronously

`Ecotone` does allow for easy change endpoint to be running synchronously or asynchronously according to current running process.

In order to run Endpoint asynchronously we need to mark it as `Asynchronous`.

```php
#[Asynchronous("orders")]
#[CommandHandler("order.place", "place_order_endpoint")
public function placeOrder(PlaceOrderCommand $command) : void
{
   // do something with $command
}
```

```php
#[Asynchronous("orders")]
#[EventHandler(endpointId: "place_order_endpoint")
public function when(OrderWasPlaced $event) : void
{
   // do something with $command
}
```

We need to add `endpointId` on our endpoint's annotation, in this case in `CommandHandler.`   
`Asynchronous` has channel name defined as `orders` we need to register such channel. In order to do it, we need to use one of the Modules, that provides pollable channels.   
At this moment following modules with pollable channels are available:

* [AMQP Support \(RabbitMQ\)](../modules/amqp-support-rabbitmq.md#message-channel)
* [DBAL Support](../modules/dbal-support.md#message-channel)

{% hint style="info" %}
Currently available Message Channels are integrated with great library [enqueue](https://github.com/php-enqueue/enqueue).
{% endhint %}

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| orders             |
+--------------------+
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| orders             |
+--------------------+
```
{% endtab %}

{% tab title="Lite" %}
```
$consumers = $messagingSystem->list()
```
{% endtab %}
{% endtabs %}

After setting up Pollable Channel we can run the endpoint:

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:run orders -vvv
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:run orders -vvv
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->run("orders");
```
{% endtab %}
{% endtabs %}

## Running configuration

### Dynamic Configuration  You may set up running configuration for given consumer while running it.

* `handledMessageLimit` - Amount of messages to be handled before stopping consumer
* `executionTimeLimit` - How long consumer should run before stopping \(milliseconds\)
* `memoryLimit` - How much memory can be consumed by before stopping consumer \(Megabytes\)
* `stopOnFailure` - Stop consumer in case of exception

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:run orders 
    --handledMessageLimit=5 
    --executionTimeLimit=1000 
    --memoryLimit=512
    --stopOnFailure
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:run orders
    --handledMessageLimit=5 
    --executionTimeLimit=1000 
    --memoryLimit=512
    --stopOnFailure
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->run(
    "orders", 
    ExecutionPollingMetadata::createWithDefault()
        ->withHandledMessageLimit(5)
        ->withMemoryLimitInMegabytes(100)
        ->withExecutionTimeLimitInMilliseconds(1000)
        ->withStopOnError(true)
);
```
{% endtab %}
{% endtabs %}

### Static Configuration

Using [Service Context ](../messaging/service-application-configuration.md)configuration for statically configuration.

```php
class Configuration
{    
    #[ServiceContext]
    public function configuration() : array
    {
        return [
            PollingMetadata::create("orders")
                 ->setErrorChannelName("errorChannel")
                 ->setInitialDelayInMilliseconds(100)
                 ->setMemoryLimitInMegaBytes(100)
                 ->setHandledMessageLimit(10)
                 ->setExecutionTimeLimitInMilliseconds(100)
        ];
    }
}
```

{% hint style="info" %}
Dynamic configuration overrides static
{% endhint %}

## Multiple Asynchronous Endpoints

Using single asynchronous channel we may register multiple endpoints.   
This allow for registering single asynchronous channel for whole Aggregate or group of related Command/Event Handlers. 

```php
#[Asynchronous("orders")]
#[EventHandler]
public function onSuccess(SuccessEvent $event) : void
{
}

#[Asynchronous("orders")]
#[EventHandler]
public function onSuccess(FailureEvent $event) : void
{
}
```

#### Asynchronous Class

You may put `Asynchronous` on the class, level so all the endpoints within a class will becomes asynchronous.

##  Intercepting asynchronous endpoint

All asynchronous endpoints are marked with special attribute`Ecotone\Messaging\Attribute\AsynchronousRunningEndpoint`   
If you want to intercept all polling endpoints you should make use of [annotation related point cut](interceptors.md#pointcut) on this.

## Running synchronously

You may register channel to be synchronously in order to run whenever the event is published.  
This can be useful in testing scenarios, where we want to test whole flow in simple manner.

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function orderChannel()
    {
        return SimpleMessageChannelBuilder::createPublishSubscribeChannel("orders");
    }
}
```

## Dropping messages coming to the channel

You may register channel to be `null channel` which means it will drop the any message it receive.  
This can be useful in testing scenarios, if we want to turn off specific functionality.

```php
class MessagingConfiguration
{
    #[ServiceContext] 
    public function orderChannel()
    {
        return SimpleMessageChannelBuilder::createNullableChannel("orders");
    }
}
```

