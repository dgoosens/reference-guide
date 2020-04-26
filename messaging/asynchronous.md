# Scheduling and Asynchronous

## Scheduling

`Ecotone` comes with support for running `period tasks` or `cron jobs` using `Scheduled.`  
`Scheduled` creates [Message](messaging-concepts/message.md) from given method and send it to `requestChannelName`.

```php
/**
 * @MessageEndpoint()
 */
class CurrencyExchanger
{
    /**
     * @Scheduled(
     *     endpointId="currencyExchanger",
     *     requestChannelName="exchange",
     *     poller=@Poller(
     *          fixedRateInMilliseconds=1000
     *     )
     * )
     */
    public function callExchange() : array
    {
        return ["currency" => "EUR", "ratio" => 1.23];
    }
}


/**
 * @CommandHandler(inputChannelName="exchange")
 */
public function exchange(ExchangeCommand $command) : void;
```

`endpointId` - `Scheduled` requires defined `endpointId,` it will be used in order to run Adapter.   
`requestChannelName` - The channel name to which [Message](messaging-concepts/message.md) should be send  
`poller` - Configuration how to execute Inbound Channel Adapter, [read more in next section](asynchronous.md#polling-metadata). This configuration tells `Ecotone` to execute Channel Adapter every second.

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:list-all-asynchronous-endpoints
+--------------------+
| Endpoint Names     |
+--------------------+
| currencyExchanger  |
+--------------------+
```
{% endtab %}

{% tab title="Lite" %}
```php
$consumers = $messagingSystem->getListOfSeparatelyRunningConsumers()
```
{% endtab %}
{% endtabs %}

After setting up Scheduled endpoint we can run the endpoint:

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:run-endpoint currencyExchanger -vvv
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->runSeparatelyRunningEndpointBy("currencyExchanger");
```
{% endtab %}
{% endtabs %}

After running`currencyExchanger` endpoint it will poll message from `callExchange`and call  Command Handler `exchange`with array payload  `["currency" => "EUR", "ratio" => 1.23]`. When the Message will arrive on the Command Handler it will be automatically converted to `ExchangeCommand.` If you want to understand how the conversion works, you may read about it in [Conversion section](conversion/).

## Polling Metadata

Polling Metadata defines how polling consumer should behave. 

```php
     @Poller(
1          cron="* * * * *",
2          initialDelayInMilliseconds=2000,
3          fixedRateInMilliseconds=1000,
4          memoryLimitInMegabytes=100,
5          handledMessageLimit=10,
6          executionTimeLimitInMilliseconds=200,
7          errorChannelName="errorChannel"
     )
```

`cron` - Defines that consumer should be called according to given cron expression  
`initialDelayInMilliseconds` - Delay after executing consumer, before consumer will start polling  
`memoryLimitInMegabytes` - Limit of RAM, before execution of consumer will be stopped.  
`handledMessageLimit` - Limit of handled messages, before execution of consumer will be stopped  
`executionTimeLimitInMilliseconds` - How long consumer should be running before it will be stopped  
`errorChannelName` - In case of failure during consumer execution, where to send exception. 

## Error Channel

Depending on consumer, exceptions during consumer execution may stop the consumer or just reject the message \(e.g. AMQP Consumer\) and consumer will continue.   
`Error Channel` can be defined in order to redirect exceptions before they will be returned to the consumer. This way we can add extra logging, store exceptions or add custom exception handling.

```php
/**
 * @MessageEndpoint()
 */
class CurrencyExchanger
{
    /**
     * @InboundChannelAdapter(
     *     endpointId="currencyExchanger",
     *     requestChannelName="exchange",
     *     poller=@Poller(
     *          fixedRateInMilliseconds=1000,
     *          errorChannelName="errorHandler"         
     *     )
     * )
     */
    public function callExchange() : array
    {
        return ["currency" => "EUR", "ratio" => 1.23];
    }
    
   /**
   * @CommandHandler(inputChannelName="exchange")
   */
   public function exchange(ExchangeCommand $command) : void
   {
      throw new \InvalidArgumentException("exchange failure");
   }
}

/**
 * @MessageEndpoint()
 */
class CustomErrorHandler
{
   /**
   * @ServiceActivator(inputChannelName="errorHandler")
   */
   public function exchange(MessagingException $exception) : void
   {
      // handle exception
   }
}
```

{% hint style="info" %}
You may wonder what is ServiceActivator. This is lower API Endpoint.   
Actually Command/Query/Event Handlers are all Service Activators.   
  
We do not want to expose such endpoint to public world, that's why we use @ServiceActivator as it not available via Command/Event/Query buses. 
{% endhint %}

## Asynchronous

`Ecotone` does allow for easy change endpoint to be running synchronously or asynchronously according to current running process.

In order to run Endpoint asynchronously we need to mark it as `@Asynchronous`.

```php
use Ecotone\Messaging\Annotation\Asynchronous;

/**
 * @Asynchronous(channelName="orders")
 * @CommandHandler(endpointId="place_order_endpoint", inputChannelName="order.place")
 */
public function placeOrder(PlaceOrderCommand $command) : void
{
   // do something with $command
}
```

We need to add endpointId on our endpoint's annotation, in this case in `@CommandHandler.`   
`@Asynchronous` has channelName defined as `orders` we need to register such channel. In order to do it, we need to use one of the Modules, that provides pollable channels.   
At this moment following modules with pollable channels are available:

* [AMQP Support \(RabbitMQ\)](../modules/amqp-support-rabbitmq.md#message-channel)
* [DBAL Support](../modules/dbal-support.md#message-channel)

{% hint style="info" %}
Currently available Message Channels are integrated with great library [enqueue](https://github.com/php-enqueue/enqueue).
{% endhint %}

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:list-all-pollable-endpoints
+--------------------+
| Endpoint Names     |
+--------------------+
| orders             |
+--------------------+
```
{% endtab %}

{% tab title="Lite" %}
```php
$consumers = $messagingSystem->getListOfSeparatelyRunningConsumers()
```
{% endtab %}
{% endtabs %}

After setting up Pollable Channel we can run the endpoint:

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:run-endpoint orders -vvv
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->runSeparatelyRunningEndpointBy("orders");
```
{% endtab %}
{% endtabs %}

### Asynchronous Class

You may put `@Asynchronous` on the class, level so all the endpoints within a class will becomes asynchronous.

### Query Handler

`Query Handler` endpoints are never registered as asynchronous.

### Multiple Asynchronous Endpoints

Using single asynchronous channel we may register multiple endpoints.   
This allow for registering single asynchronous channel for whole Aggregate or group of related Command Handlers, that should be done in order. 

### Polling Metadata for Asynchronous

As asynchronous channel can have multiple endpoints, we can't define [Polling Metadata](asynchronous.md#polling-metadata) on the specific Endpoint. That's why we need to make use of `Application Context Configuration`.  
`Application Context Configuration`is configuration done on PHP Level.  

```php
use Ecotone\Messaging\Annotation\ApplicationContext;
use Ecotone\Messaging\Annotation\Extension;

/**
 * @ApplicationContext()
 */
class Configuration
{
    /**
     * @Extension()
     */
    public function registerAsyncChannelPollingMetadata() : array
    {
        return [
            PollingMetadata::create("orders")
                ->setErrorChannelName(self::ERROR_CHANNEL)
        ];
    }
}
```

`@ApplicationContext` - Tells `Ecotone` that this class is `Ecotone` application level configuration  
`@Extension` - Marks method that extends Ecotone with specific configuration. Can return object or array of configurations.   
  
Configuration can be done for specific environment. 

```php
use Ecotone\Messaging\Annotation\ApplicationContext;
use Ecotone\Messaging\Annotation\Environment;
use Ecotone\Messaging\Annotation\Extension;

/**
 * @ApplicationContext()
 */
class Configuration
{
    /**
     * @Extension()
     * @Environment({"test"})
     */
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
If you want to intercept all polling endpoints you should make use of [annotation related point cut](interceptors.md#pointcut).  
 `@(Ecotone\Messaging\Annotation\PollableEndpoint)`

## Examples

Examples can be [find here](https://github.com/ecotoneframework/examples/tree/master/src/Scheduling) or in specific modules.

