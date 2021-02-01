# Lesson 4: Metadata and Method Invocation

{% hint style="info" %}
Not having code for _Lesson 4?_   
  
`git checkout lesson-4`
{% endhint %}

### Metadata

Message can contain headers. Headers may contain anything e.g. `currentUser`, `timestamp`, `contentType`, `messageId.` Headers are helpful, to give extra insights about the message and how it should be handled.   
In `Ecotone` headers and metadata means the same. Those terms will be used interchangeably.

We just got new requirement for our Products in Shopping System.:

`User who registered the product, should be able to change it price.` 

Let's start by adding `ChangePriceCommand.`

```php
namespace App\Domain\Product;

class ChangePriceCommand
{
    private int $productId;

    private Cost $cost;

    public function getProductId() : int
    {
        return $this->productId;
    }

    public function getCost() : Cost
    {
        return $this->cost;
    }
}
```

We will handle this `Command` in a minute. Let's first add user information for registering the product.  
We will do it, using `Meta Data`. Let's get back to our Testing Class `EcotoneQuickstart.`

```php
public function run() : void
{
    $this->commandBus->convertAndSendWithMetadata(
        "product.register",
        MediaType::APPLICATION_JSON,
        \json_encode(["productId" => 1, "cost" => 100]),
        [
            "userId" => 5
        ]
    );
            
    echo $this->queryBus->convertAndSend("product.getCost", MediaType::APPLICATION_JSON, \json_encode(["productId" => 1]));
}
```

We call a different method `convertAndSendWithMetadata,` it accepts 4th argument, which is `associative array.` Whatever we will place in here, will be available during message handling for us.  
So it's good place, to enrich the message with extra information, which we will want to use or store, but are not part of the `command/query/event`.   
Now we can change our `Product` aggregate: 

```php
/**
 * @Aggregate()
 */
class Product
{
    use WithAggregateEvents;

    /**
     * @AggregateIdentifier()
     */
    private int $productId;

    private Cost $cost;

    private int $userId;

    private function __construct(int $productId, Cost $cost, int $userId)
    {
        $this->productId = $productId;
        $this->cost = $cost;
        $this->userId = $userId;

        $this->record(new ProductWasRegisteredEvent($productId));
    }

    /**
     * @CommandHandler(inputChannelName="product.register")
     */
    public static function register(RegisterProductCommand $command, array $metadata) : self
    {
        return new self($command->getProductId(), $command->getCost(), $metadata["userId"]);
    }
```

We have added second parameter `$metadata` to our `@CommandHandler`. `Ecotone` does the job, by reading parameters and evaluating what should be injected. We will see soon, how can we take control of this process.   
  
We can add `changePrice` method now.

```php
/**
 * @CommandHandler(inputChannelName="product.changePrice")
 */
public function changePrice(ChangePriceCommand $command, array $metadata) : void
{
    if ($metadata["userId"] !== $this->userId) {
        throw new \InvalidArgumentException("You are not allowed to change the cost of this product");
    }

    $this->cost = $command->getCost();
}
```

And let's call it with incorrect `userId` and see, if we get the exception.

```php
public function run() : void
{
    $this->commandBus->convertAndSendWithMetadata(
        "product.register",
        MediaType::APPLICATION_JSON,
        \json_encode(["productId" => 1, "cost" => 100]),
        [
            "userId" => 5
        ]
    );

    $this->commandBus->convertAndSendWithMetadata(
        "product.changePrice",
        MediaType::APPLICATION_JSON,
        \json_encode(["productId" => 1, "cost" => 110]),
        [
            "userId" => 3
        ]
    );        
```

{% hint style="success" %}
Let's run our testing command:
{% endhint %}

```php
bin/console ecotone:quickstart
Running example...
Product with id 1 was registered!

InvalidArgumentException
                                                          
  You are not allowed to change the cost of this product 
```

### Method Invocation

We have been just informed, that customers are registering new products in our system.   
  
`Only administrator should be allowed to register new product`

Let's create simple `UserService` which will tell us, if specific user is administrator.   
In our testing scenario we will suppose, that only user with `id` of 1 is administrator. 

```php
namespace App\Domain\Product;

class UserService
{
    public function isAdmin(int $userId) : bool
    {
        return $userId === 1;
    }
}
```

Now we need to think where we should call our `UserService.`   
The good place for it, would not allow for any invocation of `product.register command` without being administrator, otherwise our constraint may be bypassed.  
`Ecotone` does allow for auto-wire like injection for endpoints. All services registered in Depedency Container are available.

```php
/**
 * @CommandHandler(inputChannelName="product.register")
 */
public static function register(RegisterProductCommand $command, array $metadata, UserService $userService) : self
{
    $userId = $metadata["userId"];
    if (!$userService->isAdmin($userId)) {
        throw new \InvalidArgumentException("You need to be administrator in order to register new product");
    }

    return new self($command->getProductId(), $command->getCost(), $userId);
}
```

Great, there is no way to bypass the constraint now. The `isAdmin constraint` must be satisfied in order to register new product.   
  
Let's correct our testing class.  

```php
public function run() : void
{
    $this->commandBus->convertAndSendWithMetadata(
        "product.register",
        MediaType::APPLICATION_JSON,
        \json_encode(["productId" => 1, "cost" => 100]),
        [
            "userId" => 1
        ]
    );

    $this->commandBus->convertAndSendWithMetadata(
        "product.changePrice",
        MediaType::APPLICATION_JSON,
        \json_encode(["productId" => 1, "cost" => 110]),
        [
            "userId" => 1
        ]
    );

    echo $this->queryBus->convertAndSend("product.getCost", MediaType::APPLICATION_JSON, \json_encode(["productId" => 1]));
}
```

{% hint style="success" %}
Let's run our testing command:
{% endhint %}

```php
bin/console ecotone:quickstart
Running example...
Product with id 1 was registered!
110
Good job, scenario ran with success!
```

### Injecting arguments

`Ecotone` inject arguments based on `parameter converters`.  
Parameter converters , tells `Ecotone` how to resolve specific parameter and what kind of argument it is expecting.  The one used for injecting services like the `UserService` is `@Reference` converter.  
Let's see how could we use it in our `product.register` command handler. 

Let's suppose UserService is registered under `user-service` in Dependency Container. Then we would need to set up the `@CommandHandler`like below.

```php
use Ecotone\Messaging\Annotation\Parameter\Reference;

@CommandHandler(
   inputChannelName="product.register", 
   parameterConverters={
      @Reference(parameterName="userService", referenceName="user-service")
   }
)
```

`parameterName` - Is used widely between different converters. It does tell `Ecotone` to which parameter name from the called method this converter is related.

`@Reference`- Does inject service from Dependency Container. If `referenceName,` which is name of the service in the container is not given, then it will take the class name as default.

`@Payload` - Does inject payload of the [message](../messaging/messaging-concepts/message.md). In our case it will be the command itself

`@Headers` - Does inject all headers as array.

`@Header` - Does inject single header from the [message](../messaging/messaging-concepts/message.md).    
  
There is more to be said about this, but at this very moment, it will be enough for us to know that such possibility exists in order to continue.  
You may read more detailed description in [Method Invocation section.](../messaging/conversion/method-invocation.md)   


### Default Converters

`Ecotone`, if parameter converters are not passed provides default converters.   
First parameter is always `@Payload.`   
The second parameter, if is `array` then `@Headers` converter is taken, otherwise if class type hint is provided for parameter, then `@Reference` converter is picked.   
If we would want to manually configure parameters for `product.register` Command Handler, then it would look like this:

```php
/**
 * @CommandHandler(
 *     inputChannelName="product.register",
 *     parameterConverters={
 *          @Payload(parameterName="command"),
 *          @Headers(parameterName="metadata"),
 *          @Reference(parameterName="userService")
 *     }
 * )
 */
public static function register(RegisterProductCommand $command, array $metadata, UserService $userService) : self
{
    $userId = $metadata["userId"];
    if (!$userService->isAdmin($userId)) {
        throw new \InvalidArgumentException("You need to be administrator in order to register new product");
    }

    return new self($command->getProductId(), $command->getCost(), $metadata["userId"]);
}
```

{% hint style="success" %}
Great, we have just finished Lesson 4!

In this Lesson we learned about using Meta Data to provide extra information to our Message.  
Besides we took a look on how arguments are injected into endpoint and how we can make use of it.  
  
Now we will learn about Interceptors
{% endhint %}

