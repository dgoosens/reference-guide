# Quick Start - DDD CQRS PHP

Below quick starts will teach you how to use specific `Ecotone's` module in _5 minutes_.  
So you may quick start integration with your existing project or build new one with ease.    
If you want to know in depth with `Ecotone`, consider going through [Tutorial](../tutorial-php-ddd-cqrs-event-sourcing/).

## Show me examples!

Do you want to see examples straight away?

### - [CQRS](php-cqrs.md) \(Command Query Responsibility Segregation\)

### [- Event Handling](php-event-handling.md)

## Before we start Quick Start

Choose your preferred platform `Symfony`  `Laravel` or `Lite (Only Ecotone)` and install accordingly with help of [Installation Section](../install-php-service-bus.md).

### Access To Buses

In tutorial we will be using Command/Query/Event buses. It's important to know, how to access them depending on chosen platform. 

{% tabs %}
{% tab title="Symfony" %}
```php
There is no need for any extra configuration
Command/Query/Event Buses are automatically registered in Symfony Dependency Container
and are available using auto-wire system

As an example, we may inject Command into Symfony Controller:

use Ecotone\Modelling\CommandBus;

class UserController
{
    private CommandBus $commandBus;

    public function __contruct(CommandBus $commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function register(Request $request): Response
    {
        $this->commandBus->send(new RegisterUser($request->get("name")));
        
        return new Response();
    }
}
```
{% endtab %}

{% tab title="Laravel" %}
```php
There is no need for any extra configuration
Command/Query/Event Buses are automatically registered in Laravel Depedency Container
and are available using auto-wire system

As an example, we may inject Command Bus into Laravel Controller:

use Ecotone\Modelling\CommandBus;

class UserController extends Controller
{
    private CommandBus $commandBus;

    public function __contruct(CommandBus $commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function register(Request $request)
    {
        $this->commandBus->send(new RegisterUser($request->input("name")));

        return response('');
    }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
1. You may access Command/Query/Event buses using getGatewayByName method. 

/** @var \Ecotone\Modelling\CommandBus $commandBus */
$commandBus = $messagingSystem->getGatewayByName(\Ecotone\Modelling\CommandBus::class);

2. If your container implements GatewayAwareContainer, then all buses will be
available for you via injection. 

class SomethingUsingCommandBus
{
    private CommandBus $commandBus;

    public function __contruct(CommandBus $commandBus)
    {
        $this->commandBus = $commandBus;
    }
}

```
{% endtab %}
{% endtabs %}



