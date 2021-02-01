# Lesson 1: Messaging Concepts

{% hint style="info" %}
Not having code for _Lesson 1?_ 

`git checkout lesson-1`
{% endhint %}

## Key concepts / background

In this first lesson, we will learn fundamental blocks in messaging architecture and we will start building back-end for Shopping System using CQRS.   


_Ecotone_ from the ground is built around messaging to provide a simple model for building integration solutions while maintaining the separation of concerns that is essential for producing maintainable, testable code. To achieve that, fundamental messaging blocks are implemented using [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com) \(EIP\). In order to use higher level of abstraction, CQRS and DDD support, we need to briefly get known with those patterns. This will help us understand, how specific parts of _Ecotone_ does work together. 

{% hint style="info" %}
_`Ecotone`_ is heavily inspired by [_Spring Integration_](https://spring.io/projects/spring-integration)_,_ [_Axon Framework_](https://docs.axoniq.io/reference-guide/) _and_ [_NServiceBus_](https://particular.net/nservicebus)_._
{% endhint %}

### Message

![](../.gitbook/assets/message.jpg)

A message is a generic wrapper for any PHP type \(scalar, object, compound\) with metadata used by the framework while handling that object.  
The payload can be of any type, and the headers \(metadata\) hold commonly required information such as ID, timestamp and framework specific information. Headers are also used for passing values to and from connected transports.   
Developers can also store any arbitrary key-value pairs in the headers, to pass needed meta information.

```php
interface Message
{
    public function getPayload();

    public function getHeaders() : MessageHeaders;
}
```

### Message Channel

![](../.gitbook/assets/message-channel-connection.svg)

_Message channel_ abstracts communication between components. It does allow for sending and receiving messages.  
A message channel may follow either point-to-point or publish-subscribe semantics.   
With a point-to-point channel, only one consumer can receive each message sent to the channel.   
Publish-subscribe channels, broadcast each message to all subscribers on the channel. 

_Pollable channels_ extends Message Channels with capability of buffering Messages within a queue. The advantage of buffering is that it allows for throttling the inbound messages and preventing of message loss. 

### Message Endpoint

![](../.gitbook/assets/endpoint-1.jpg)

Message Endpoints are consumers and producers of messages. Consumer are not necessary asynchronous, as you may build synchronous flow, compound of multiple endpoints.   
You will not have to implement them directly, as you should not even have to build messages and invoke or receive message directly from the [Message channel](lesson-1-messaging-concepts.md#message-channel). Instead you will be able to focus on your specific domain model with with an implementation based on plain PHP objects. By providing declarative configuration, you can "connect” your domain-specific code to the messaging infrastructure provided by `Ecotone`. 

{% hint style="info" %}
If you are familiar with Symfony Messager/Simplebus, for now you can think of Endpoint as a _Message Handler_, that can be connected to asynchronous or synchronous transport. 
{% endhint %}

### Messaging Gateway

![](../.gitbook/assets/gateway_execution.svg)

The Messaging Gateway encapsulates messaging-specific code \(The code required to send or receive a [Message](lesson-1-messaging-concepts.md#message)\) and separates it from the rest of the application code.  
It does build from domain specific objects a [Message](lesson-1-messaging-concepts.md#message) that is send via [Message channel](lesson-1-messaging-concepts.md#message-channel).   
To not have dependency on the `Ecotone` _API_ — including the gateway class, `Ecotone` provides the Gateway as interface. Framework generates a proxy for any interface and internally invokes the gateway methods. By using dependency injection, you can then expose the interface to your business methods.

{% hint style="info" %}
Command/Query/Event buses are implemented using Messaging Gateway.
{% endhint %}

{% hint style="success" %}
Great, now when we know fundamental blocks of `Ecotone` and _Messaging Architecture_, we can start implementing our Shopping System!   
If you did not understand something, do not worry, we will see how does it apply in practice in next step. 
{% endhint %}

## Implementation

Do you remember this command from [Setup part](before-we-start-tutorial.md#setup-for-tutorial)?

```php
bin/console ecotone:quickstart
"Running example...
Hello World
Good job, scenario ran with success!"
```

If yes and this command does return above output, then we are ready to go.

{% tabs %}
{% tab title="Symfony" %}
{% code title="https://github.com/ecotoneframework/quickstart-symfony/blob/lesson-1/src/EcotoneQuickstart.php" %}
```php
Go to "src/EcotoneQuickstart.php"
```
{% endcode %}
{% endtab %}

{% tab title="Laravel" %}
{% code title="https://github.com/ecotoneframework/quickstart-laravel/blob/master/app/EcotoneQuickstart.php" %}
```php
Go to "app/EcotoneQuickstart.php"
```
{% endcode %}
{% endtab %}
{% endtabs %}

this method will be run, whenever we execute_`ecotone:quickstart`_.   
This class is auto-registered using auto-wire system, both [Symfony](https://symfony.com/doc/current/service_container/autowiring.html) and [Laravel](https://laravel.com/docs/7.x/container) provides this great feature.   
Thanks to that, we will not need to write any service registration configuration during this tutorial and we will be able to focus fully on what can `Ecotone` provide to us. 

```php
<?php

namespace App;

class EcotoneQuickstart
{
    public function run() : void
    {
        echo "Hello World";
    }
}
```

### Command Handler - Endpoint

We will create our first `Command Handler` endpoint connected to `Point To Point Channel`.  
`Command Handler` is place where we will put our business logic.   
Let's create namespace `App\Domain\Product` and inside `RegisterProductCommand,` command for registering new product:

```php
<?php

namespace App\Domain\Product;

class RegisterProductCommand
{
    private int $productId;

    private int $cost;

    public function __construct(int $productId, int $cost)
    {
        $this->productId = $productId;
        $this->cost = $cost;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }

    public function getCost() : int
    {
        return $this->cost;
    }
}
```

{% hint style="info" %}
In the tutorial we are using PHP 7.4, feel free to use PHP 7.3, but remember to document your code.  
PHP &gt;= 7.4

```php
private int $productId;
```

PHP &lt;= 7.3

```php
/**
 * @var int
 */
private $productId;
```

Describing types, will help us in later lessons with automatic conversion. Just remember right now, that it's worth to keep the types defined.
{% endhint %}

Let's register a Command Handler now by creating class `App\Domain\Product\ProductService`

```php
<?php

namespace App\Domain\Product;

use Ecotone\Messaging\Annotation\ClassReference;
use Ecotone\Modelling\Annotation\CommandHandler;

class ProductService
{
    private array $registeredProducts = [];
    
    /**
     * @CommandHandler()
     */
    public function register(RegisterProductCommand $command) : void
    {
        $this->registeredProducts[$command->getProductId()] = $command->getCost();
    }
}
```

First thing worth noticing is `@CommandHandler`.   
This annotation marks our `register` method in ProductService as an [Endpoint](lesson-1-messaging-concepts.md#message-endpoint), from that moment it can be found by `Ecotone.`

`Ecotone` will read method declaration and base on the first parameter type hint will know that this `@CommandHandler` is responsible for handling `RegisterProductCommand.` 

{% hint style="info" %}
`@ClassReference` is a special [Annotation](https://www.doctrine-project.org/projects/doctrine-annotations/en/1.8/index.html)  it informs `Ecotone`how this service is registered in `Depedency Container`. As a default it takes the class name, which is compatible with auto-wiring system. If `ProductService` would be registered in `Dependency Container` as `productService`, we would do this:

```php
/**
 * @ClassReference("productService")
 */
class ProductService
```
{% endhint %}

### Query Handler - Endpoint

We also need possibility to query our `ProductService` for registered products.  
This is the role of `Query Handlers`. They do query the state and return it to us.    
Let's starts with `GetProductPriceQuery` _class._ This _query_ will tell us what is the price of specific product.

```php
<?php

namespace App\Domain\Product;

class GetProductPriceQuery
{
    private int $productId;

    public function __construct(int $productId)
    {
        $this->productId = $productId;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }
}
```

We also need Handler for this query. Let's add `Query Handler` __to the `ProductService`

```php
<?php

namespace App\Domain\Product;

use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\Annotation\QueryHandler;

class ProductService
{
    private array $registeredProducts = [];

    /**
     * @CommandHandler()
     */
    public function register(RegisterProductCommand $command) : void
    {
        $this->registeredProducts[$command->getProductId()] = $command->getCost();
    }

    /**
     * @QueryHandler()
     */
    public function getPrice(GetProductPriceQuery $query) : int
    {
        return $this->registeredProducts[$query->getProductId()];
    }
}
```

{% hint style="info" %}
Some CQRS frameworks expects Handlers be defined as a class, not method. This is somehow limiting and producing a lot of boilerplate. `Ecotone` does allow for full flexibility, if you want to have only one handler per class, so be it, if more just annotate next methods.
{% endhint %}

{% tabs %}
{% tab title="Laravel" %}
```php
# As default auto wire of Laravel creates new service instance each time 
# service is requested from Depedency Container, we need to register 
# ProductService as singleton.

# Go to bootstrap/QuickStartProvider.php and register our ProductService

namespace Bootstrap;

use App\Domain\Product\ProductService;
use Illuminate\Support\ServiceProvider;

class QuickStartProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(ProductService::class, function(){
            return new ProductService();
        });
    }
(...)
```
{% endtab %}

{% tab title="Symfony" %}
```php
Everything is set up by the framework, please continue...
```
{% endtab %}
{% endtabs %}

### Command and Query Bus - Gateways

It's time to call our endpoints.  
You may remember that [endpoints](lesson-1-messaging-concepts.md#message-endpoint) need to be connected using [Message Channels](lesson-1-messaging-concepts.md#message-channel) and we did not do anything like this yet. Thankfully `Ecotone` does create synchronous channels for us, so we don't need to bother about it. 

{% hint style="info" %}
In case of Query And Command Handlers, point to point channels are created automatically.  
You will learn how to easily replace them with asynchronous channels in next lessons.
{% endhint %}

We need to create [`Message`](lesson-1-messaging-concepts.md#message) and send it to correct [`Message Channel`](lesson-1-messaging-concepts.md#message-channel).   
In order to do it we will use [Messaging Gateway](lesson-1-messaging-concepts.md#messaging-gateway). Message Gateways are responsible for creating `Message` from given parameters and send it to the correct `channel`.  
For sending Commands we will use `Command Bus Gateway`.  
For sending Queries we will use `Query Bus Gateway`.   
  
Let's inject and call Query and Command bus into EcotoneQuickstart class. 

```php
<?php

namespace App;

use App\Domain\Product\GetProductPriceQuery;
use App\Domain\Product\RegisterProductCommand;
use Ecotone\Modelling\CommandBus;
use Ecotone\Modelling\QueryBus;

class EcotoneQuickstart
{
    private CommandBus $commandBus;
    private QueryBus $queryBus;

// 1
    public function __construct(CommandBus $commandBus, QueryBus $queryBus)
    {
        $this->commandBus = $commandBus;
        $this->queryBus = $queryBus;
    }

    public function run() : void
    {
// 2    
        $this->commandBus->send(new RegisterProductCommand(1, 100));
// 3
        echo $this->queryBus->send(new GetProductPriceQuery(1));
    }
}
```

1. Gateways are auto registered in Dependency Container and available for auto-wire. `Ecotone` comes with few set up Gateways. Command and Query buses are available instantly to you. You will be able to extend them or create your own ones, if needed.
2. We are sending command`RegisterProductCommand` to the `@CommandHandler` we registered before. 
3. Same as above, but in that case we are sending query `GetProductPriceQuery` to the `@QueryHandler`

If you run our testing command now, you should see the result. 

```php
bin/console ecotone:quickstart
Running example...
100
Good job, scenario ran with success!
```

### Event Publishing

We want to notify, when new product is registered in the system.   
In order to do it, we will make use of `Event Bus Gateway` which can publish events.  
Let's start by creating `ProductWasRegisteredEvent.`

```php
<?php

namespace App\Domain\Product;

class ProductWasRegisteredEvent
{
    private int $productId;

    public function __construct(int $productId)
    {
        $this->productId = $productId;
    }

    public function getProductId() : int
    {
        return $this->productId;
    }
}
```

{% hint style="info" %}
As you can see `Ecotone` does not really care what class Command/Query/Event is. It does not require to implement any interfaces neither prefix or suffix the class name.  
In fact commands, queries and events can be of any type and we will see it in next Lessons.  
In the tutorial we do use Command/Query/Event suffixes to clarify the distinction.
{% endhint %}

Let's inject `EventBus` into our `@CommandHandler` in order to publish `ProductWasRegisteredEvent`.

```php
use Ecotone\Modelling\EventBus;

/**
 * @CommandHandler()
 */
public function register(RegisterProductCommand $command, EventBus $eventBus) : void
{
    $this->registeredProducts[$command->getProductId()] = $command->getCost();

    $eventBus->send(new ProductWasRegisteredEvent($command->getProductId()));
}
```

You may wonder, how `EventBus` is injected into the Command Handler's method.  
`Ecotone` does control method invocation for [endpoints](lesson-1-messaging-concepts.md#message-endpoint), if you have type hinted for specific class, framework will look in Dependency Container for specific service in order to inject it automatically. It does work similar to auto-wire system. If you want to know more, check chapter about [Method Invocation](../messaging/conversion/method-invocation.md).

Now, when our event is published, whenever new product is registered, we want to listen for it and notify. Let's create new class and annotate method with `@EventHandler`. 

```php
<?php

namespace App\Domain\Product;

use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\EventHandler;

/**
 * @MessageEndpoint()
 */
class ProductNotifier
{
    /**
     * @EventHandler() // 1
     */
    public function notifyAbout(ProductWasRegisteredEvent $event) : void
    {
        echo "Product with id {$event->getProductId()} was registered!\n";
    }
}
```

1. `@EventHandler` tells `Ecotone` to handle specific event based on declaration type hint, just like with `@CommandHandler.` 

{% hint style="info" %}
You may use interfaces or abstract classes for your first parameter type hint. Ecotone will resolve connection between published event and Event Handler.
{% endhint %}

If you run our testing command now, you should see the result. 

```php
bin/console ecotone:quickstart
Running example...
Product with id 1 was registered!
100
Good job, scenario ran with success!
```

{% hint style="success" %}
Great, we have just finished Lesson 1.   
In this lesson we have learned basic of Messaging and CQRS.   
  
We are ready for Lesson 2!
{% endhint %}

{% page-ref page="lesson-2-tactical-ddd.md" %}

