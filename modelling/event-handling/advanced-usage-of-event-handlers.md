# Advanced Usage of Event Handlers

Ecotone allows for handling events with Aggregates, which lies down a lot of possibility for developers, to make simple solutions for complex problems.

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

    #[EventHandler(dropMessageOnNotFound: true)] 
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

You can find more information in [Saga section](../saga.md).

