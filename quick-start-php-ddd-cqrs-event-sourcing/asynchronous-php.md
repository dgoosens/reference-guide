---
description: Running the code asynchronously
---

# Asynchronous PHP

## What is Asynchronous Processing and What Does It Give

[Read more about Asynchronous in PHP and Ecotone](https://blog.ecotone.tech/asynchronous-php/)

## To The Code!

Let's create `Event` _Order was placed_.

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

```php
class NotificationService
{
    const ASYNCHRONOUS_MESSAGES = "asynchronous_messages";

    #[Asynchronous("asynchronous_messages")]
    #[EventHandler(endpointId:"notifyAboutNeworder")]
    public function notifyAboutNewOrder(OrderWasPlaced $event) : void
    {
        echo "Handling asynchronously: " . $event->getProductName() . "\n";
    }
}
```

Let's `Ecotone` that we want to run this Event Handler Asynchronously using [RabbitMQ](https://www.rabbitmq.com/)

```php
class Configuration
{
    #[ServiceContext]
    public function enableRabbitMQ()
    {
        return AmqpBackedMessageChannelBuilder::create(NotificationService::ASYNCHRONOUS_MESSAGES);
    }
}
```

## Running The Example

```php
$eventBus->publish(new OrderWasPlaced(1, "Milk"));

# Running asynchronous consumer
$messagingSystem->run("asynchronous_messages");
```

## Implementation Using Lite

[https://github.com/ecotoneframework/quickstart-examples/tree/master/Asynchronous](https://github.com/ecotoneframework/quickstart-examples/tree/master/Asynchronous)

