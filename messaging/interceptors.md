# Interceptors

`Ecotone` provide possibility to handle [cross cutting concerns](https://en.wikipedia.org/wiki/Cross-cutting_concern) via `Interceptors`.   
`Interceptor` as name the suggest, intercepts the process of handling the message.   
You may enrich the [message](messaging-concepts/message.md), stop or modify usual processing cycle, call some shared functionality, add additional behavior to existing code without modifying the code itself. 

{% hint style="info" %}
If you are familiar with [Aspect Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_programming) or Middleware pattern \(used in most of PHP CQRS frameworks\) you may find some similarities.
{% endhint %}

## Interceptor

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class AdminVerificator
{
    /**
     * @Before(
     *     precedence=0,  
     *     pointcut="Order\Domain\*"
     * )
     */
    public function isAdmin(array $payload, array $headers) : void
    {
        if ($headers["executorId"] != 1) {
            throw new \InvalidArgumentException("You need to be administrator in order to register new product");
        }
    }
}
```

`@MethodInterceptor`- marks class as `Interceptor`, to be found by `Ecotone.`

`@Before` - Type of Interceptor more about it [Interceptor Types section](interceptors.md#interceptor-types)

### Precedence

Precedence defines ordering of called interceptors. The lower the value is, the quicker Interceptor will be called. It's safe to stay with range between -1000 and 1000, as numbers bellow -1000 and higher than 1000 are used by `Ecotone.`   
The precedence is done within a specific [interceptor type](interceptors.md#interceptor-types). 

### Pointcut

Every interceptor has `Pointcut` attribute, which describes for specific interceptor, which endpoints it should intercept.

* `CLASS_NAME` - indicates specific class or interface which should be intercepted
* `@(CLASS_NAME)` - Indicating all [Endpoints](messaging-concepts/message-endpoint/) having `CLASS_NAME` annotation on method or class level 
* `NAMESPACE*` - Indicating all [Endpoints](messaging-concepts/message-endpoint/) starting with namespace prefix e.g. `App\Domain\*`
* `expression||expression` - Indicating one expression or another e.g. `Product\*||Order\*` 

### Parameter Converters

You may pass optional parameter converters, if needed. If you want to read about parameter converters go to [Method Invocation section](conversion/method-invocation.md#parameter-converter-types).

## Interceptor Types

There are four types of interceptors. Each interceptor has it role and possibilities.   
Interceptors are called in following order:

* Presend 
* Before
* Around
* After

## Before Interceptor

### Exceptional Interceptor 

`Before interceptor` is called before endpoint is executed.   
Before interceptors can used in order to `stop the flow`, `throw an exception` or `enrich the` [`Message.`](messaging-concepts/message.md)  
  
We will intercept Command Handler with verification if executor is an administrator.

Let's start by creating `Annotation` called `IsAdministrator` in new namepace.

```php
/**
 * @Annotation
 */
class RequireAdministrator {}
```

{% hint style="info" %}
Ecotone does use Doctrine Annotations. If you are not familiar with the concept of annotation, you may take a brew introduction [on their website](https://www.doctrine-project.org/projects/doctrine-annotations/en/1.6/index.html).
{% endhint %}

Let's create our first `Before Interceptor.` 

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class AdminVerificator
{
    /**
     * @Before(pointcut="@(RequireAdministrator)")
     */
    public function isAdmin(array $payload, array $headers) : void
    {
        if ($headers["executorId"] != 1) {
            throw new \InvalidArgumentException("You need to be administrator in order to register new product");
        }
    }
}
```

We are using in here [Pointcut](interceptors.md#pointcut) here which is looking for `@RequireAdministrator` annotation in each of registered [Endpoints](messaging-concepts/message-endpoint/).  
The `void return type` is expected in here. It tells `Ecotone`that, this Before Interceptor is not modifying the Message and message will be passed through. The message flow however can be interrupted by throwing exception.

Now we need to annotate our Command Handler:

```php
/**
 * @CommandHandler()
 * @RequireAdministrator()
 */
public function changePrice(ChangePriceCommand $command) : void
{
   // do something with $command
}
```

Whenever we call our command handler, it will be intercepted by AdminVerificator now.  
There is one thing worth to mention. Our `Command Handler` is using `ChangePriceCommand`class and our `AdminVerificator interceptor` is using `array $payload`. They are both the same payload of the [Message](messaging-concepts/message.md), but converted in the way [Endpoint](messaging-concepts/message-endpoint/) expected. 

### Payload Enriching Interceptor

If return type is `not void` ****new modified based on previous Message will be created from the returned type.   
We will enrich [Message](messaging-concepts/message.md) payload with timestamp.

```php
/**
 * @Annotation
 */
class AddTimestamp {}
```

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class TimestampService
{
    /**
     * @Before(pointcut="@(AddTimestamp)")
     */
    public function add(array $payload) : array
    {
        return array_merge($payload, ["timestamp" => time()]);
    }
}
```

```php
class ChangePriceCommand
{
    private int $productId;
    
    private int $timestamp;
}

/**
 * @CommandHandler()
 * @AddTimestamp()
 */
public function changePrice(ChangePriceCommand $command) : void
{
   // do something with $command and timestamp
}
```

### Header Enriching Interceptor

Suppose we want to add executor Id, but as this is not part of our Command, we want add it to our [Message](messaging-concepts/message.md) Headers.

```php
/**
 * @Annotation
 */
class AddExecutor {}
```

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class TimestampService
{
    /**
     * @Before(
     *  pointcut="@(AddExecutor)",
     *  changeHeaders=true
     * )
     */
    public function add() : array
    {
        return ["executorId" => 1];
    }
}
```

If return type is `not void` ****new modified based on previous Message will be created from the returned type. If we additionally add `changeHeaders=true`it will tell Ecotone, that we we want to modify Message headers instead of payload. 

```php
/**
 * @CommandHandler()
 * @AddExecutor()
 */
public function changePrice(ChangePriceCommand $command, array $metadata) : void
{
   // do something with $command and executor id $metadata["executorId"]
}
```

### Message Filter Interceptor

Use `Message Filter`, to eliminate undesired messages based on a set of criteria.  
This can be done by returning null from interceptor, if the flow should proceed, then payload should be returned.

```php
/**
 * @Annotation
 */
class SendNotificationOnlyIfInterested {}
```

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class NotificationFilter
{
    /**
     * @Before(
     *  pointcut="@(SendNotificationOnlyIfInterested)",
     *  changeHeaders=true
     * )
     */
    public function filter(PriceWasChanged $event) : ?array
    {
        if ($this->isInterested($event) {
           return $event; // flow proceeds 
        }
        
        return null;  // message is eliminated, flow stops.
    }
    
    private function isInterested(PriceWasChanged $event) : bool
    {
       // some logic which decides, if X is interested in notification
    }
}
```

If return type is `not void` ****new modified based on previous Message will be created from the returned type. If we additionally add `changeHeaders=true`it will tell Ecotone, that we we want to modify Message headers instead of payload. 

```php
/**
 * @EventHandler()
 * @SendNotificationOnlyIfInterested()
 */
public function sendNewPriceNotification(ChangePriceCommand $event) : void
{
   // do something with $event
}
```

## Around Interceptor

The `Around Interceptor` have access to actual `Method Invocation.`This does allow for starting some procedure and ending after the invocation is done.  At this moment all conversions are done, so we can't convert payload to different type. 

`Around interceptor`is a good place for handling transactions or logic shared between different endpoints, that need to access invoked object. 

```php
use Ecotone\Messaging\Annotation\Interceptor\Around;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;
use Ecotone\Messaging\Handler\Processor\MethodInvoker\MethodInvocation;

/**
 * @MethodInterceptor()
 */
class TransactionInterceptor
{
    /**
     * @Around(pointcut="Ecotone\Modelling\CommandBus")
     */
    public function transactional(MethodInvocation $methodInvocation)
    {
        $this->connection->beginTransaction();
        try {
            $result = $methodInvocation->proceed();

            $this->connection->commit();
        }catch (\Throwable $exception) {
            $this->connection->rollBack();

            throw $exception;
        }

        return $result;
    }
}
```

As we used `Command Bus` interface as  pointcut, we told `Ecotone` that it should intercept `Command Bus Gateway.`  Now whenever we will call any method on Command Bus, it will be intercepted with transaction.  
  
The other powerful use case for Around Interceptor is intercepting Aggregate.   
Suppose we want to verify, if executing user has access to the Aggregate.

```php
/**
 * @Aggregate()
 * @IsOwnerOfPerson
 */
class Person
{
   private string $personId;

   /**
   *  @CommandHandler()
   */
   public function changeAddress(ChangeAddress $command) : void
   {
      // change address
   }
   
   public function hasPersonId(string $personId) : bool
   {
      return $this->personId === $personId;
   }
}
```

We have placed `@IsOwnerOfPerson` annotation as the top of class. For interceptor pointcut it means, that each endpoint defined in this class should be intercepted. No need to add it on each Command Handler now.

```php
/**
 * @Annotation
 */
class IsOwnerOfPerson {}
```

```php
use Ecotone\Messaging\Annotation\Interceptor\Around;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;
use Ecotone\Messaging\Handler\Processor\MethodInvoker\MethodInvocation;

/**
 * @MethodInterceptor()
 */
class IsOwnerVerificator
{
    /**
     * @Around(pointcut="@(IsOwnerOfPerson)")
     */
    public function isOwner(MethodInvocation $methodInvocation, Person $person, array $metadata)
    {
        if (!$person->hasPersonId($metadata["executoId"]) {
           throw new \InvalidArgumentException("No access to do this action!");
        }
        $result = $methodInvocation->proceed();
    }
}
```

## After Interceptor

`After interceptor` is called after endpoint execution has finished.   
It does work exactly the same as [`Before Interceptor.`](interceptors.md#before-interceptor)  
After interceptor can used to for example to enrich `QueryHandler` result.

```php
namespace Order\ReadModel;

class OrderService
{

   /**
   * @QueryHandler()
   */
   public function getOrderDetails(GetOrderDetailsQuery $query) : array
   {
      return ["orderId" => $query->getOrderId()]
   }
}   
```

```php
use Ecotone\Messaging\Annotation\Interceptor\Before;
use Ecotone\Messaging\Annotation\Interceptor\MethodInterceptor;

/**
 * @MethodInterceptor()
 */
class AddResultSet
{
    /**
     * @After(
     *  pointcut="Order\ReadModel\*"
     * )
     */
    public function add(array $payload) : array
    {
        return ["result" => $payload];
    }
}
```

We will intercept all endpoints within Order\ReadModel namespace, by adding result coming from the endpoint under `result` key.

## Presend Interceptor

`Before Send Interceptor`  is called before Message is actually send to the channel.  
In synchronous channel there is no difference between `Before` and `Before Send.`   
The difference is seen when the channel is [asynchronous](asynchronous.md).

####  Before Interceptor

![](../.gitbook/assets/before-interceptor.svg)

Before Interceptor is called after message is sent to the channel, before execution of Endpoint.

####  Presend Interceptor

![](../.gitbook/assets/presend-interceptor.svg)

Presend Interceptor is called exactly before message is sent to the channel.   
  
`Presend Interceptor` can be used for example, when Command Bus is called from HTTP Controller.   
Then we may want to verify if data is correct and if not filter out the Message, or we may want to check, if user has enough permissions to do the action and throw an exception before message will go asynchronous. 

## Examples

Examples can be [find here](https://github.com/ecotoneframework/examples/tree/master/src/Modelling/Intercepting).

