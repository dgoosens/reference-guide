# Asynchronous Handling

## Asynchronous

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

We need to add endpointId on our endpoint's annotation, in this case in `CommandHandler.`   
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

{% tab title="Lite" %}
```php
$consumers = $messagingSystem->getListOfAsynchronouslyRunningConsumers()
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

{% tab title="Lite" %}
```php
$messagingSystem->runAsynchronouslyRunningEndpoint("orders");
```
{% endtab %}
{% endtabs %}

### Asynchronous Class

You may put `Asynchronous` on the class, level so all the endpoints within a class will becomes asynchronous.

### Query Handler

`Query Handler` endpoints are never registered as asynchronous.

### Multiple Asynchronous Endpoints

Using single asynchronous channel we may register multiple endpoints.   
This allow for registering single asynchronous channel for whole Aggregate or group of related Command Handlers, that should be done in order. 

### Polling Metadata for Asynchronous

As asynchronous channel can have multiple endpoints, we can't define [Polling Metadata](../messaging/scheduling.md#polling-metadata) on the specific Endpoint. That's why we need to make use of `Application Context Configuration`.  
`Application Context Configuration`is configuration done on PHP Level.  

```php
class Configuration
{    
    #[ServiceContext]
    public function registerAsyncChannelPollingMetadata() : array
    {
        return [
            PollingMetadata::create("orders")
                ->setErrorChannelName(self::ERROR_CHANNEL)
        ];
    }
}
```

`@Extension` - Marks method that extends Ecotone with specific configuration. Can return object or array of configurations.   
  
Configuration can be done for specific environment. 

```php
class Configuration
{
    #[Extension]
    #[Environment(["test"]) 
    public function registerAsyncChannelPollingMetadata() : array
    {
        return [
            PollingMetadata::create("orders")
                ->setExecutionTimeLimitInMilliseconds(1)
                ->setHandledMessageLimit(1)
        ];
    }
}
```

##  Intercepting asynchronous endpoint

All asynchronous endpoints are marked with special annotation`Ecotone\Messaging\Annotation\PollableEndpoint`   
If you want to intercept all polling endpoints you should make use of [annotation related point cut](../messaging/interceptors.md#pointcut).  
 `Ecotone\Messaging\Annotation\PollableEndpoint`

