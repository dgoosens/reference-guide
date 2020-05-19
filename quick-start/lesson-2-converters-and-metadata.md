# Lesson 2: Tactical DDD

{% hint style="info" %}
Not having code for _Lesson 2?_ 

`git checkout lesson-2`
{% endhint %}

### Aggregate

An Aggregate is an entity or group of entities that is always kept in a consistent state.    
Aggregates are very explicitly present in the Command Model, as that is where change is initiated and business behaviour is placed.

Let's create our first _Aggregate_ `Product.`

```php
namespace App\Domain\Product;

use Ecotone\Modelling\Annotation\Aggregate;
use Ecotone\Modelling\Annotation\AggregateIdentifier;
use Ecotone\Modelling\Annotation\CommandHandler;
use Ecotone\Modelling\Annotation\QueryHandler;

/**
 * @Aggregate() // 1
 */
class Product
{
    /**
     * @AggregateIdentifier() // 2
     */
    private int $productId;

    private int $cost;

    private function __construct(int $productId, int $cost)
    {
        $this->productId = $productId;
        $this->cost = $cost;
    }

    /**
     * @CommandHandler() // 3
     */
    public static function register(RegisterProductCommand $command) : self
    {
        return new self($command->getProductId(), $command->getCost());
    }

    /**
     * @QueryHandler() // 4
     */
    public function getCost(GetProductPriceQuery $query) : int
    {
        return $this->cost;
    }
}
```

1. `@Aggregate` annotation marks class to be known as Aggregate
2. `@AggregateIdentififer` marks properties as identifiers of specific Aggregate instance. Each _Aggregate_ must contains at least one identifier. 
3. `@CommandHandler` enables command handling on specific method just as we did in [Lesson 1](lesson-1-messaging-concepts.md).  If method is static, it's treated as [factory method](https://en.wikipedia.org/wiki/Factory_method_pattern) and must return new aggregate instance. Rule applies as long as we do [State-Stored Aggregate](../modelling/command-handling/state-stored-aggregate.md#state-stored-aggregate) instead of [Event Sourcing Aggregate](../modelling/command-handling/event-sourcing-aggregate.md).
4. `@QueryHandler` enables query handling on specific method just as we did in Lesson 1.

{% hint style="info" %}
If you want to known more details about _Aggregate_ start with chapter [State-Stored Aggregate](../modelling/command-handling/state-stored-aggregate.md#state-stored-aggregate)
{% endhint %}

Now remove `App\Domain\Product\ProductService` as it contains handlers for same command and query classes.   
Before we will run our test scenario, we need to register `Repository`.

{% hint style="info" %}
Usually you will mark `services` as Query Handlers not `aggregates. Ecotone`does not block possibility to place Query Handler on _Aggregate_. It's up to you, where do you want to place Query Handler.
{% endhint %}

### Repository

Repositories are used for retrieving and saving the aggregate to persistent storage.   
We will build in memory implementation, as this will be enough for us.

```php
namespace App\Domain\Product;

use Ecotone\Modelling\Annotation\Repository;
use Ecotone\Modelling\StandardRepository;

/**
 * @Repository() // 1
 */
class InMemoryProductRepository implements StandardRepository // 2
{
    /**
     * @var Product[]
     */
    private $products = [];

    // 3
    public function canHandle(string $aggregateClassName): bool
    {
        return $aggregateClassName === Product::class;
    }

    // 4
    public function findBy(string $aggregateClassName, array $identifiers): ?object
    {
        if (!array_key_exists($identifiers["productId"], $this->products)) {
            return null;
        }

        return $this->products[$identifiers["productId"]];
    }

    // 5
    public function save(array $identifiers, object $aggregate, array $metadata, ?int $expectedVersion): void
    {
        $this->products[$identifiers["productId"]] = $aggregate;
    }
}
```

1. `@Repository` annotation marks class to be known to `Ecotone` as Repository.
2. We need to implement some methods in order to allow `Ecotone`, retrieve and save Aggregate. Based on implemented interface, `Ecotone` knowns, if _Aggregate_ is state-stored or event sourced.  
3. `canHandle` tells which classes can be handled by this specific repository
4. `findBy`  return found aggregate instance or null. As there may be more, than single indentifier per aggregate, identifiers are array.
5. `save` saves passed aggregate instance. You do not need to bother right what is `$metadata` and `$expectedVersion`

{% hint style="info" %}
If you want to known more details about _Repository_ start with chapter [Repository](../modelling/command-handling/repository.md)
{% endhint %}

{% tabs %}
{% tab title="Laravel" %}
```php
# As default auto wire of Laravel creates new service instance each time 
# service is requested from Depedency Container, we need to register 
# ProductService as singleton.

# Go to bootstrap/QuickStartProvider.php and register our ProductService

namespace Bootstrap;

use App\Domain\Product\InMemoryProductRepository;
use Illuminate\Support\ServiceProvider;

class QuickStartProvider extends ServiceProvider
{
    public function register()
    {
        $this->app->singleton(InMemoryProductRepository::class, function(){
            return new InMemoryProductRepository();
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

{% hint style="success" %}
Let's run our testing command:

```php
bin/console ecotone:quickstart
Running example...
100
Good job, scenario ran with success!
```
{% endhint %}

Have you noticed, what are we missing here? Our `Event Handler` was not called, as we do not publish `ProductWasRegistered` event at this moment. 

### Event Publishing

In order to automatically publish events recorded within Aggregate, we need to add method annotated with `@AggregateEvents.` This will tell `Ecotone` where to get the events from.  
  
`Ecotone` comes with default implementation, that can be used as trait `WithAggregateEvents`.

```php
use Ecotone\Modelling\WithAggregateEvents;

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

    private int $cost;

    private function __construct(int $productId, int $cost)
    {
        $this->productId = $productId;
        $this->cost = $cost;

        $this->record(new ProductWasRegisteredEvent($productId));
    }
(...)
```

{% hint style="info" %}
You may implement your own method for returning events, if you do not want to couple with framework.
{% endhint %}

{% hint style="success" %}
Let's run our testing command:
{% endhint %}

```php
bin/console ecotone:quickstart
Running example...
Product with id 1 was registered!
100
Good job, scenario ran with success!
```

{% hint style="success" %}
Congratulations, we have just finished Lesson 2.  
In this lesson we learnt how to make use of Aggregates and Repositories.  
  
Now we will learn about Converters and Metadata
{% endhint %}

