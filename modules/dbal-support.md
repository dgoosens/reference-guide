# DBAL Support

## Installation

```php
composer require ecotone/dbal
```

## Configuration

In order to use `Dbal Support` we need to add `ConnectionFactory` to our `Dependency Container.` 

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
    Enqueue\Dbal\DbalConnectionFactory:
        class: Enqueue\Dbal\DbalConnectionFactory
        arguments:
            - "pgsql://user:password@host:5432/db_name"
```
{% endtab %}

{% tab title="Laravel" %}
```php
# Register Service in Provider

use Enqueue\Dbal\DbalConnectionFactory;

public function register()
{
     $this->app->singleton(DbalConnectionFactory::class, function () {
         return new DbalConnectionFactory('pgsql://user:password@host:5432/db_name');
     });
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
We register our `DbalConnectionFactory` under the class name `Enqueue\Dbal\DbalConnectionFactory`. This will help Ecotone resolve it automatically, without any additional configuration.
{% endhint %}

If we want to make use of existing connection using `Manager Registry`, we can do it this way

{% tabs %}
{% tab title="Symfony" %}
```php
# config/services.yaml
# You need to have RabbitMQ instance running on your localhost, or change DSN
    Enqueue\Dbal\DbalConnectionFactory:
        class: Enqueue\Dbal\ManagerRegistryConnectionFactory
        arguments:
            - "@doctrine.orm.entity_manager"
            - 
                connection_name: "default"
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Register Manager Registry under `DbalConnectionFactory`, if you want to make use of auto configuration.   
Otherwise you will need to tell Message Channel, Transactions the name of `Connection Factory`.
{% endhint %}

## Message Channel

To create `Dbal Backed Channel`, we need to create `Application Context.` 

```php
use Ecotone\Amqp\AmqpBackedMessageChannelBuilder;
use Ecotone\Messaging\Annotation\ApplicationContext;
use Ecotone\Messaging\Annotation\Extension;

/**
 * @ApplicationContext()
 */
class MessagingConfiguration
{
    /**
     * @Extension()
     */
    public function orderChannel()
    {
        return DbalBackedMessageChannelBuilder::create("orders");
    }
}
```

Now `orders` channel will be available in our Messaging System. 

## Transactions

`Ecotone Dbal` comes with support for transactions.    
To enable transactions on specific endpoint, mark it with `Ecotone\Dbal\DbalTransaction\DbalTransaction` annotation.

```php
    /**
     * @CommandHandler()
     * @DbalTransaction()
     */
    public function sellProduct(SellProduct $command) : void
    {
        // do something with $command
    }
```

If you want to enable for all [Asynchronous Endpoints](../quick-start/lesson-6-scheduling-and-asynchronous.md) or specific for Command Bus. You may use of `ApplicationContext.`

```php
use Ecotone\Dbal\Configuration\DbalConfiguration;
use Ecotone\Messaging\Annotation\ApplicationContext;
use Ecotone\Messaging\Annotation\Extension;

/**
 * @ApplicationContext()
 */
class ChannelConfiguration
{
    /**
     * @Extension()
     */
    public function registerTransactions() : array
    {
        return [
            DbalConfiguration::createWithDefaults()
                ->withTransactionOnAsynchronousEndpoints(true)
                ->withTransactionOnCommandBus(true)
        ];
    }

}
```

## Examples

Examples can be [find here](https://github.com/ecotoneframework/examples/tree/master/src/Dbal/Async).

