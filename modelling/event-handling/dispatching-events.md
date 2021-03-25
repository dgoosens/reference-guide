---
description: Events PHP
---

# Dispatching Events

Previous pages provide the background on how to handle event messages in your application. The dispatching process is the starting point for event message.  
You will mostly send events after successfully handling [Command Message](../command-handling/), examples will base on that assumption.

{% hint style="info" %}
You can inject EventBus and send events wherever necessary. Ecotone tries not to impose any  specific solutions.
{% endhint %}

## Event Bus

`Event Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface EventBus
{
    1 public function send(object $event);

    2 public function sendWithMetadata(object $event, array $metadata);

    3 public function convertAndSend(string $name, string $sourceMediaType, $data);

    4 public function convertAndSendWithMetadata(string $name, string $sourceMediaType, $data, array $metadata);
}
```

### Send method

Event is routed to the Handler by class type.

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\EventBus;

class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
//     handle closing ticket

       $eventBus->send(new TicketWasClosed($command->getTicketId())); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
use Ecotone\Modelling\Annotation\EventHandler;

class NotificationService
{   
    /**
    * @EventHandler()
    */
    public function closeTicket(TicketWasClosed $event) : void
    {
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

### SendWithMetadata

Does allow for passing `extra meta information`, that can be used on targeted `Event Handler`.

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\EventBus;

class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
//     handle closing ticket

       $eventBus->sendWithMetadata(new TicketWasClosed($command->getTicketId()), ["executorUsername" => "someUsername"]); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
use Ecotone\Modelling\Annotation\EventHandler;

class NotificationService
{   
    /**
    * @EventHandler()
    */
    public function closeTicket(TicketWasClosed $event, array $metadata) : void
    {
        // you can use $metadata["executorUsername"]
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

### ConvertAndSend

Is used with `Command Handlers,`routed by name and converted using [Converter](../../messaging/conversion/) if needed.  
Sending events by name instead of class type, may be found useful in integration with external application, when events are in different Media Type than PHP class.  

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\EventBus;

class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
//     handle closing ticket

       $eventBus->convertAndSend(
          "ticket.wasClosed", 
          "application/json", 
          '{"ticketId": 123}'
       ); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
use Ecotone\Modelling\Annotation\EventHandler;

class NotificationService
{   
    /**
    * @EventHandler(listenTo="ticket.wasClosed")
    */
    public function closeTicket(TicketWasClosed $event) : void
    {
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
JSON will be automatically converted to specific `class type hinted` in method declaration of Event Handler. You could also use in here simple `array` if you have `JSON` to `array` [Converter](../../messaging/conversion/) or a `string`, if you like to receive `JSON string`.
{% endhint %}

### ConvertAndSendWithMetadata

Same as [convertAndSend](dispatching-events.md#convertandsend) with possibility to pass `Metadata.`

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\EventBus;

class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, EventBus $eventBus) : void
    {
//     handle closing ticket

       $eventBus->convertAndSendWithMetadata(
          "ticket.wasClosed", 
          "application/json", 
          '{"ticketId": 123}',
          ["executorUsername" => "someUsername"]
       ); 
    }   
}
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Event Handler" %}
```php
use Ecotone\Modelling\Annotation\EventHandler;

class NotificationService
{   
    /**
    * @EventHandler(listenTo="ticket.wasClosed")
    */
    public function closeTicket(TicketWasClosed $event, array $metadata) : void
    {
        // you can use $metadata["executorUsername"]
        // notify about closing the ticket
    }   
}
```
{% endtab %}
{% endtabs %}

## Lazy Event Bus

`Lazy Event Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface LazyEventBus
{
    1 public function send(object $event);

    2 public function sendWithMetadata(object $event, array $metadata);
}
```

### Send method

The difference between `EventBus` and `LazyEventBus` is the moment of `EventHandler` invocation.  
`EventBus` sends Event right away to all awaiting `EventHandlers`.   
`LazyEventBus` registers the Event and waiting for `Command Handler` to finish. When the Command Handling is done, then it publish previously registered events.    
Below we can comparison in execution order:

{% tabs %}
{% tab title="Lazy Event Bus" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\LazyEventBus;

class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, LazyEventBus $eventBus) : void
    {
       echo "starting command handler";
       
       $eventBus->send(new TicketWasClosed($command->getTicketId())); 
       
       echo "finished command handler";
    }   
}

class NotificationService
{   
    /**
    * @EventHandler()
    */
    public function closeTicket(TicketWasClosed $event) : void
    {
        echo "notification for event handler"
    }   
}

-----
1. "starting command handler"
2. "finished command handler"
3. "notification for event handler"
```
{% endtab %}

{% tab title="Event Bus" %}
```php
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\LazyEventBus;

/**
 *  @MessageEndpoint()
 */
class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command, LazyEventBus $eventBus) : void
    {
       echo "starting command handler";
       
       $eventBus->send(new TicketWasClosed($command->getTicketId())); 
       
       echo "finished command handler";
    }   
}

/**
 * @MessageEndpoint()
 */
class NotificationService
{   
    /**
    * @EventHandler()
    */
    public function closeTicket(TicketWasClosed $event) : void
    {
        echo "notification for event handler"
    }   
}

-----
1. "starting command handler"
2. "notification for event handler"
3. "finished command handler"
```
{% endtab %}
{% endtabs %}

### SendWithMetadata

Does work the same as [Send](dispatching-events.md#send-method-1) with possibility to send `Metadata`. Usage is no different than with [standard Event Bus](dispatching-events.md#sendwithmetadata).

## Publishing from Aggregate

The easiest way to publish events from aggregate would be to inject `EventBus` into it's `Command Handling` method. That would works, however there is a better solution.   
`Ecotone` does provide possibility to automatically gather events from `Aggregate` and publish them using `EventBus.`   


### State-Stored Aggregate

  
To tell `Ecotone`, which method it should use for retrieving Event objects when using [State-Stored Aggregate](../command-handling/state-stored-aggregate.md#state-stored-aggregate) mark method containing events with annotation `@AggregateEvents.` After handling `Command` or `Event`  on `Aggregate` events will be published. 

```php
/**
 * @return object[]
 * @AggregateEvents()
 */
public function getRecordedEvents() : array
```

 If you do want to bother with implementation you can make use of trait `WithAggregateEvents`

```php
use Ecotone\Modelling\Annotation\AggregateEvents;
```

If you want to record event for publication just use `record`method.

```php
$this->record(new OrderWasPlaced());
```

Events will be automatically retrieved and published after handling current Command or Event.

### Event Sourcing Aggregate

When using [Event Sourcing Aggregate]() you do not need to do anything extra. Each method should return events after handling, those events will automatically published using `Event Bus.`

```php
public function assignWorker(AssignWorkerCommand $command) : array
```

