---
description: Scheduling PHP
---

# Scheduling

## Scheduling

`Ecotone` comes with support for running `period tasks` or `cron jobs` using `Scheduled.`  
`Scheduled` creates [Message](../messaging/messaging-concepts/message.md) from given method and send it to `requestChannelName`.

```php
class CurrencyExchanger
{
    #[Scheduled(requestChannelName: "exchange", endpointId: "currencyExchanger")] 
    #[Poller(fixedRateInMilliseconds=1000)]
    public function callExchange() : array
    {
        return ["currency" => "EUR", "ratio" => 1.23];
    }
}

#[CommandHandler("exchange")] 
public function exchange(ExchangeCommand $command) : void;
```

`endpointId` - `Scheduled` requires defined `endpointId,` it will be used in order to run Adapter.   
`requestChannelName` - The channel name to which [Message](../messaging/messaging-concepts/message.md) should be send  
`poller` - Configuration how to execute Inbound Channel Adapter, [read more in next section](scheduling.md#polling-metadata). This configuration tells `Ecotone` to execute Channel Adapter every second.

{% tabs %}
{% tab title="Symfony" %}
```php
console ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| currencyExchanger  |
+--------------------+
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| currencyExchanger  |
+--------------------+
```
{% endtab %}

{% tab title="Lite" %}
```php
$consumers = $messagingSystem->list()
```
{% endtab %}
{% endtabs %}

After setting up Scheduled endpoint we can run the endpoint:

{% tabs %}
{% tab title="Symfony" %}
```php
console ecotone:run currencyExchanger -vvv
```
{% endtab %}

{% tab title="Laravel" %}
```
artisan ecotone:run currencyExchanger -vvv
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->run("currencyExchanger");
```
{% endtab %}
{% endtabs %}

After running`currencyExchanger` endpoint it will poll message from `callExchange`and call  Command Handler `exchange`with array payload  `["currency" => "EUR", "ratio" => 1.23]`. When the Message will arrive on the Command Handler it will be automatically converted to `ExchangeCommand.` If you want to understand how the conversion works, you may read about it in [Conversion section](../messaging/conversion/).

## Polling Metadata

Polling Metadata defines how polling consumer should behave. 

```php
     #[Poller(
1          cron="* * * * *",
2          initialDelayInMilliseconds=2000,
3          fixedRateInMilliseconds=1000,
4          memoryLimitInMegabytes=100,
5          handledMessageLimit=10,
6          executionTimeLimitInMilliseconds=200,
7          errorChannelName="errorChannel"
     )]
```

`cron` - Defines that consumer should be called according to given cron expression  
`initialDelayInMilliseconds` - Delay after executing consumer, before consumer will start polling  
`memoryLimitInMegabytes` - Limit of RAM, before execution of consumer will be stopped.  
`handledMessageLimit` - Limit of handled messages, before execution of consumer will be stopped  
`executionTimeLimitInMilliseconds` - How long consumer should be running before it will be stopped  
`errorChannelName` - In case of failure during consumer execution, where to send exception. 

