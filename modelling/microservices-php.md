---
description: Distributed Services PHP
---

# Microservices PHP

## Distribution

`Ecotone` provides support for communication in distributed architecture.   
The support covers sending command directly to specific service \(application\) or publishing events, that specific services may listen for. 

## Configuration

In order for `Ecotone` how to route messages you need to register Service Name \(Application Name\).

* [Symfony Service Name Configuration](../modules/symfony-ddd-cqrs-event-sourcing.md#servicename)
* [Laravel Service Name Configuration](../modules/laravel-ddd-cqrs-event-sourcing.md#servicename)
* [Ecotone Lite Service Name Configuration](../modules/ecotone-lite.md#servicename) 

### Modules Providing Support

* [RabbitMQ Module](../modules/amqp-support-rabbitmq.md#distributed-publisher-and-consumer)

## Distribution Bus

Distribution Bus is [Message Gateway](../messaging/messaging-concepts/messaging-gateway.md) just like CommandBus or EventBus.   
The bus is responsible for distribution of your command and events. 

  
Depending on your [Module configuration](microservices-php.md#configuration) you can inject it by `class name` or `specific reference name`.

```php
interface DistributedBus
{
// Command distribution

    public function sendCommand(string $destination, string $routingKey, string $command, string $sourceMediaType = MediaType::TEXT_PLAIN, array $metadata = []) : void;

    public function convertAndSendCommand(string $destination, string $routingKey, object|array $command, array $metadata = []) : void;

// Event distribution

    public function publishEvent(string $routingKey, string $event, string $sourceMediaType = MediaType::TEXT_PLAIN, array $metadata = []) : void;

    public function convertAndPublishEvent(string $routingKey, object|array $event, array $metadata) : void;
}
```

## Sending Distributed Commands

* `destination` - is a [Service Name](microservices-php.md#configuration) of targeted Service
* `routingKey` - is a routing key under which CommandHandler is registered on targeted Service

```php
public function changeBillingDetails(DistributedBus $distributedCommandBus)
{
    $distributedCommandBus->sendCommand(
        "billing", // destination
        "billing.changeDetails", // routingKey
        '["personId":"123","billingDetails":"01111"]',
        "application/json"
    );
}
```

## Consuming Distributed Commands

### Register distributed command handler

On the consumer side register distributed command handler

```php
#[Distributed]
#[CommandHandler("billing.changeDetails")]
public function changeBillingDetails(ChangeBillingDetails $command) : void
{
    // do something with billing details
}
```

{% hint style="info" %}
To expose command handler for service distribution mark it with `Distributed` attribute.
{% endhint %}

### Run the consumer

Run consumer for your registered distributed consumer. It will be available under your [Service Name](microservices-php.md#configuration)

List:

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| billings           |
+--------------------+
```
{% endtab %}

{% tab title="Laravel" %}
```php
artisan ecotone:list
+--------------------+
| Endpoint Names     |
+--------------------+
| billing            |
+--------------------+
```
{% endtab %}

{% tab title="Lite" %}
```
$consumers = $messagingSystem->list()
```
{% endtab %}
{% endtabs %}

Run it:

{% tabs %}
{% tab title="Symfony" %}
```php
bin/console ecotone:run billing -vvv
```
{% endtab %}

{% tab title="Laravel" %}
```
artisan ecotone:run billing -vvv
```
{% endtab %}

{% tab title="Lite" %}
```php
$messagingSystem->run("billing");
```
{% endtab %}
{% endtabs %}

## Publishing Distributed Events

* `routingKey` - is a routing key under which event will be published

```php
public function changeBillingDetails(DistributedBus $eventBus)
{
    $eventBus->publishEvent(
        "billing.detailsWereChanged", // routingKey
         '{"personId":123,"billingDetails":0001}',
         "application/json" 
    );
}
```

## Consuming Distributed Events

```php
#[Distributed]
#[EventHandler("billing.detailsWereChanged")]
public function registerTicket(BillingDetailsWereChanged $event) : void
{
    // do something with event
}
```

{% hint style="info" %}
To start listening for distributed events under specific key, provide `Distributed` attribute.
{% endhint %}

### Run the consumer

Run it the same as for [distributed command bus](microservices-php.md#run-the-consumer).

