---
description: Events PHP
---

# Handling Events

## Registering Class Based Event Handler

In order to handle events published within a system we can use  `#[EventHandler]` with combination of specific `class`.

```php
class Notifier
{
    #[EventHandler] 
    public function notify(AppointmentWasCreatedEvent $event) : void
    {
        // do something with $event
    }
}
```

In order to listen for event we need to mark method as `EventHandler.` This work exactly the same as [External Command Handlers.](../command-handling/external-command-handlers.md)

Whenever `AppointmentWasCreatedEvent` will be published our `notify` method will be called,

### Event Handling Interfaces and Abstract Classes

```php
#[EventHandler]
public function notify(AppointmentRelatedEvent $event) : void
{
   // do something with $event
}
```

We also can tell `Ecotone` that we want to handle all events implementing specific Interface or extending specific abstract class. In that case all we need to do it is to type hint for this interface or abstract class. 

### Event Handling Union Classes

```php
#[EventHandler]
public function notify(AppointmentWasScheduled|AppointmentWasCancelled $event) : void
{
   // do something with $event
}
```

Ecotone provides possibility to listen for multiple events with same method, based on Union Types.

### Handling all published events

```php
#[EventHandler]
public function storeLog(object $event) : void
{
   // do something with $event
}
```

If we will type hint for `object` then this `Event Handler` will be called for any event published in the system.

## Registering Name Based Events

There may be a situations when events will arrive from different systems. We may not have access to specific class or we may not want to share events a cross applications. In that case we can listen for events with specific name. 

```php
class EmailService
{
    #[EventHandler("billing.customer_was_invoiced")] 
    public function notifyAboutGeneratedInvoiced(array $event) : void
    {
        // do something with $event
    }
}
```

This `Event Handler` listen for `billing.customer_was_invoiced.` 

{% hint style="info" %}
How to publish named events, you may see in [Dispatching Events section](dispatching-events.md)
{% endhint %}

```php
class EmailService
{
    #[EventHandler("billing.customer_was_invoiced")] 
    public function notifyAboutGeneratedInvoiced(CustomerWasInvoiced $event) : void
    {
        // do something with $event
    }
}
```

In above case EventHandler will be registered only for event routed by `billing.customer_was_invoiced.` If you want to register it for class based routing also, then you should do it like below:

```php
class EmailService
{
    #[EventHandler("billing.customer_was_invoiced")] 
    #[EventHandler()]
    public function notifyAboutGeneratedInvoiced(CustomerWasInvoiced $event) : void
    {
        // do something with $event
    }
}
```

### Handling group of named events

In order to handle more than one named event, we can make use of regex like star **`*`**

```php
#[EventHandler("shop.order.*")
public function notifyAboutGeneratedInvoiced(array $event) : void
{
    // do something with $event
}
```

This `Event Handler` will listen for all events beginning with `shop.order` prefix. So `shop.order.was_started` `shop.order.was_finished` are going to be handled by this [Endpoint](../../messaging/messaging-concepts/message-endpoint/).  


You may put the `*` expression anywhere you like

* `shop.*.processed`
* `*`

If we put only the star, then it will handle all named events. 

