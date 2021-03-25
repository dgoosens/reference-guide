---
description: Saga PHP Process Manager
---

# Saga

Saga is responsible for coordination of long running processes. It can store information about what happened and make a decision what to do in the right time.

## Saga

In `Ecotone, Saga is POPO Aggregate,`as Aggregate comes with a lot of functionality, that is used in typical Saga implementation. This provides a lot of flexibility, as Aggregate can combine behaviour of Saga and vice versa, when needed. 

{% hint style="info" %}
It's really up to you, if you want to distinct Sagas from Aggregates or combine them, `Ecotone` does not try to impose the solution.
{% endhint %}

### Storing Saga's State

Depending on the [Repository](command-handling/repository.md) implementation, you may store Saga as event stream or store whole current state. 

### Handling Saga

```php
#[Aggregate]
class OrderFulfillment
{
    #[AggregateIdentifier] 
    private string $orderId;

    private bool $isFinished;
   
    private function __construct(string $orderId)
    {
        $this->orderId = $orderId;
    }

    #[EventHandler] 
    public static function start(OrderWasPlacedEvent $event) : self
    {
        return new self($event->getOrderId());
    }

    #[EventHandler] 
    public function whenPaymentWasDone(PaymentWasFinishedEvent $event, CommandBus $commandBus) : self 
    {
       if ($this->isFinished) {
           return;
       }
    
        $this->isFinished = true;
        $commandBus->send(new ShipOrderCommand($this->orderId));
    }
}
```

`Aggregate` - Saga can stated-stored [Aggregate](command-handling/state-stored-aggregate.md) or [Event Sourced Aggregate]()  
`EventHandler` - We mark method to be called, when specific event happens. 

* `start` - is `factory method`and should construct new instance `OrderFulfillment.`Depending on need you may construct differently as [Event Sourced Aggregate]().
* `paymentWasDone` - Is called when `PaymentWasFinishedEvent` event is published. We have injected `CommandBus` into the method in order to finish process by sending `ShipOrderCommand.`  We could also publish event instead.

### Event Correlation

As Saga is identified by identifier, just like an Aggregate the events need to be correlated with specific instance.  
When we do have event like `PaymentWasFinishedEvent` we need to tell `Ecotone` which instance of `OrderFulfillment` it should be retrieve from [Repository](command-handling/repository.md) and call method on.

This is done automatically, when property name in `Event` is the same as property marked as `@AggregateIdentifier` in aggregate. 

```php
class PaymentWasFinishedEvent
{
    private string $orderId;
}
```

If the property name is different we need to give `Ecotone` a hint, how to correlate identifiers. 

```php
class SomeEvent
{
    #[TargetAggregateIdentifier("orderId")] 
    private string $purchaseId;
}
```

In other scenario, when there is no property to correlate, we can make use of `Before` or `Presend` [Interceptors](interceptors.md) to enrich event's metadata with required identifier.  
Suppose the orderId identifier is available in metadata under key orderNumber, then we can tell Endpoint to use the mapping.

```php
#[EventHandler(identifierMetadataMapping: ["orderId" => "orderNumber"])]
public function failPayment(PaymentWasFailedEvent $event, CommandBus $commandBus) : self 
{
   // do something with $event
}
```

### Unordered Events

In the [previous example](saga.md#handling-saga) we have assumed, that the first event we will receive is `OrderWasPlacedEvent` and the second which finishes the Saga is `PaymentWasFinishedEvent.`   
It it's always risky to make such assumptions, especially, when events comes from different systems.  
What we could do instead, is to expect them to come in different order and handle it gracefully.

```php
#[Aggregate] 
class OrderFulfillment
{
    #[AggregateIdentifier] 
    private string $orderId;

    private bool $isFinished;

    private function __construct(string $orderId)
    {
        $this->orderId = $orderId;
    }

    #[EventHandler("whenOrderWasPlaced")]
    public static function startByPlacedOrder(OrderWasPlacedEvent $event) : self
    {
        return new self($event->getOrderId());
    }

    #[EventHandler("whenPaymentWasDone")]
    public static function startByPaymentFinished(PaymentWasFinished $event) : self 
    {
        return new self($event->getOrderId());
    }

    #[EventHandler("whenOrderWasPlaced")]
    public function whenOrderWasPlaced(OrderWasPlacedEvent $event, CommandBus $commandBus) : self
    {
        if ($this->isFinished) {
            return;
        }

        $this->isFinished = true;
        $commandBus->send(new ShipOrderCommand($this->orderId));
    }
    
    #[EventHandler("whenPaymentWasDone")] 
    public function whenPaymentWasDone(PaymentWasFinishedEvent $event, CommandBus $commandBus) : self
    {
        if ($this->isFinished) {
            return;
        }

        $this->isFinished = true;
        $commandBus->send(new ShipOrderCommand($this->orderId));
    }
}
```

```php
#[EventHandler("whenOrderWasPlaced")]
public static function startByPlacedOrder(OrderWasPlacedEvent $event) : self

#[EventHandler("whenOrderWasPlaced")]
public function whenOrderWasPlaced(OrderWasPlacedEvent $event, CommandBus $commandBus) : self
```

If you look closely, you will see that those the factory method `startByPlacedOrder` and action method `whenOrderWasPlaced` are handling the same event.   
If that's the case, Ecotone will verify, before calling factory method, if the aggregate exists and if so, will reroute the event to the action method.   
  
This solution will prevent us from depending on the order of events, without introducing routing functionality into our business code.

### Ignoring Events

There may be situations, when we will want to handle events, only if Saga already started. 

```php
#[Aggregate] 
class OrderFulfillment
{
    #[AggregateIdentifier] 
    private string $customerId;

    private function __construct(string $customerId)
    {
        $this->customerId = $customerId;
    }

    #[EventHandler] 
    public static function start(ReceivedGreatCustomerBadge $event) : void
    {
        return new self($event->getCustomerId());
    }

    #[EventHandler(dropMessageOnNotFound: true)]
    public function whenNewOrderWasPlaced(OrderWasPlaced $event, CommandBus $commandBus) : void 
    {
        $commandBus->send(new PromotionCode($this->customerId));
    }
}
```

We want to send promotion code to the customer, if he received great customer badge, but if not we do nothing. 

```php
EventHandler(dropMessageOnNotFound=true)
```

If this saga instance will be not found, then this event will be dropped and will not call `whenNewOrderWasPlaced` method.

{% hint style="info" %}
Options we used in here, can also be applied to Command Handlers
{% endhint %}

