# Event Handling

## Description

In order to notify that some event happened in the system, we may use `event publishing and handling.`  

## To The Code!

Let's create `Event` that the Order was placed.

```php
class OrderWasPlaced
{
    private string $orderId;
    private string $productName;

    public function __construct(string $orderId, string $productName)
    {
        $this->orderId = $orderId;
        $this->productName = $productName;
    }

    public function getOrderId(): string
    {
        return $this->orderId;
    }

    public function getProductName(): string
    {
        return $this->productName;
    }
}
```

 And Event Handler that will be listening to the `OrderWasPlaced`.  
Multiple `handlers` can listen to specific `event`.

```php
class NotificationService
{
    #[EventHandler]
    public function notifyAboutNewOrder(OrderWasPlaced $event) : void
    {
        echo $event->getProductName() . "\n";
    }
}
```

## Running The Example

```php
$eventBus->publish(new OrderWasPlaced(1, "Milk"));
```

## Implementation Using Lite

[https://github.com/ecotoneframework/quickstart-examples/tree/master/src/EventHandling](https://github.com/ecotoneframework/quickstart-examples/tree/master/src/EventHandling)

