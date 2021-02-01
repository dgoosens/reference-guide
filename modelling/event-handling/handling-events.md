# Handling Events

## Registering Class Based Event Handler

In order to handle events published within a system we can use  `@EventHandler` with combination of specific `class`.

```php
use Ecotone\Modelling\Annotation\EventHandler;

class Notifier
{
    /**
     * @EventHandler()
     */
    public function notify(AppointmentWasCreatedEvent $event) : void
    {
        // do something with $event
    }
}
```

In order to listen for event we need to mark method as `@EventHandler.` This work exactly the same as [External Command Handlers.](../command-handling/external-command-handlers.md)

Whenever `AppointmentWasCreatedEvent` will be published our `notify` method will be called,

### Event Handling Interfaces and Abstract Classes

```php
/**
* @EventHandler()
*/
public function notify(AppointmentRelatedEvent $event) : void
{
   // do something with $event
}
```

We also can tell `Ecotone` that we want to handle all events implementing specific Interface or extending specific abstract class. In that case all we need to do it is to type hint for this interface or abstract class. 

### Handling all published events

```php
/**
* @EventHandler()
*/
public function storeLog(object $event) : void
{
   // do something with $event
}
```

If we will type hint for `object` then this `Event Handler` will be called for any event published in the system.

## Registering Name Based Events

There may be a situations when events will arrive from different systems. We may not have access to specific class or we may not want to share events a cross applications. In that case we can listen for events with specific name. 

```php
use Ecotone\Modelling\Annotation\EventHandler;

class EmailService
{
    /**
     * @EventHandler(listenTo="billing.customer_was_invoiced")
     */
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

### Handling group of named events

In order to handle more than one named event, we can make use of regex like star **`*`**

```php
    /**
     * @EventHandler(listenTo="shop.order.*")
     */
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

