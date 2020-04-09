---
description: Start using Ecotone
---

# Quick Start

## Install for Symfony

1. Use composer in order to download `Ecotone Symfony Bundle`

{% hint style="success" %}
composer require [ecotone/](https://packagist.org/packages/ecotone/)symfony-bundle
{% endhint %}

If you're using _`Symfony Flex`_,  bundle will auto-configure. 

2. Register bundle, if needed

{% hint style="success" %}
new Ecotone\SymfonyBundle\EcotoneSymfonyBundle::class =&gt; \['all' =&gt; true\]
{% endhint %}

## Install for Laravel

1. Use composer in order to download `Ecotone Laravel`

{% hint style="success" %}
composer require [ecotone/](https://packagist.org/packages/ecotone/)laravel
{% endhint %}

Provider should be automatically registered.

2. Register provider, if needed

```php
'providers' => [
    \Ecotone\Laravel\EcotoneProvider::class
],
```

## Install Lite \(No framework\)

1. Use composer in order to download `Ecotone`

{% hint style="success" %}
composer require ecotone/ecotone
{% endhint %}

    2. Boostrap `Ecotone`

```php
use Ecotone\Lite\EcotoneLiteConfiguration;
use Ecotone\Lite\InMemoryPSRContainer;
use Ecotone\Messaging\Config\ApplicationConfiguration;

$rootCatalog = "/var/www/html"; // path to root of your project, where composer.json exists
$namespacesToUse = ["Example\Modelling\EventSourcing"]; // list of namespaces, that should be boostraped by Ecotone
$psr11CompatibleContainer = InMemoryPSRContainer::createFromObjects([TicketRepository::createEmpty()]); // you may use existing in memory implemantation for testing purposes

$messagingSystem = EcotoneLiteConfiguration::createWithConfiguration(
    $rootCatalog,
    $psr11CompatibleContainer,
    ApplicationConfiguration::createWithDefaults()
        ->withNamespaces($namespacesToUse)
);
```



{% hint style="info" %}
Ecotone is heavily based on annotations to avoid coupling domain with framework code.  
  
PHPStorm has great plugin, which allow auto-complete for annotations: [https://plugins.jetbrains.com/plugin/7320-php-annotations](https://plugins.jetbrains.com/plugin/7320-php-annotations/)
{% endhint %}

