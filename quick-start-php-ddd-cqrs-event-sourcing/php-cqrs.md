---
description: Command Query Responsibility Segregation PHP
---

# CQRS PHP

## What is CQRS and What Does It Give

[Read Blog Post about CQRS in PHP and Ecotone](https://blog.ecotone.tech/cqrs-in-php/)

## To The Code!

#### Registering Command Handlers

Let's create `PlaceOrder` `Command` that will place an order in our system.

```php
class PlaceOrder
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

And `Command Handler` that will handle this Command

```php
use Ecotone\Modelling\Attribute\CommandHandler;

class OrderService
{
    private array $orders;

    #[CommandHandler]
    public function placeOrder(PlaceOrder $command) : void
    {
        $this->orders[$command->getOrderId()] = $command->getProductName();
    }
}
```

#### Registering Query Handlers

Let's define `GetOrder` `Query` that will find our placed order.

```php
class GetOrder
{
    private string $orderId;

    public function __construct(string $orderId)
    {
        $this->orderId = $orderId;
    }

    public function getOrderId(): string
    {
        return $this->orderId;
    }
}
```

And `Query Handler`that will handle this query

```php
use Ecotone\Modelling\Attribute\CommandHandler;
use Ecotone\Modelling\Attribute\QueryHandler;

class OrderService
{
    private array $orders;

    #[CommandHandler]
    public function placeOrder(PlaceOrder $command) : void
    {
        $this->orders[$command->getOrderId()] = $command->getProductName();
    }

    #[QueryHandler]
    public function getOrder(GetOrder $query) : string
    {
         if (!array_key_exists($query->getOrderId(), $this->orders)) {
             throw new \InvalidArgumentException("Order was not found " . $query->getOrderId());
         }

         return $this->orders[$query->getOrderId()];
    }
}
```

## Running The Example

```php
$commandBus->send(new PlaceOrder(1, "Milk"));

echo $queryBus->send(new GetOrder(1));
```

## Implementation Using Lite

[https://github.com/ecotoneframework/quickstart-examples/tree/master/src/CQRS](https://github.com/ecotoneframework/quickstart-examples/tree/master/CQRS)

