---
description: 'Installing Ecotone for Symfony, Laravel or stand alone'
---

# Installation

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

### Make use of prepared setup

{% hint style="success" %}
git clone [https://github.com/ecotoneframework/lite-setup](https://github.com/ecotoneframework/lite-setup)
{% endhint %}

### Wire manually

1. In order to start we have to have composer.json with PSR-4 or PSR-0 autoload setup.  Create `composer.json` in root catalog with `autoload for src` catalog.  And add `Ecotone`

{% hint style="success" %}
composer require ecotone/ecotone
{% endhint %}

    2. As `Ecotone` require access to Dependency Container, you have to choose one in order to run the example. As `Ecotone` may register classes for easier usage, we will make our container implement `GatewayAwareContainer` to access them.

{% hint style="success" %}
Create `console.php` and set up your container \(look \#13 line\)
{% endhint %}

```php
use Ecotone\Lite\EcotoneLiteConfiguration;
use Ecotone\Lite\GatewayAwareContainer;
use Ecotone\Messaging\Config\ServiceConfiguration;

$rootCatalog = __DIR__;
require $rootCatalog . "/vendor/autoload.php";

$container = new class() implements GatewayAwareContainer {
    private Container $container;

    public function __construct()
    {
        $this->container =  // @TODO set up your container here
    }

    public function get($id)
    {
        return $this->container->get($id);
    }

    public function has($id)
    {
        return $this->container->has($id);
    }

    public function addGateway(string $referenceName, object $gateway): void
    {
        $this->container->set($referenceName, $gateway);
    }
};

$messagingSystem = EcotoneLiteConfiguration::createWithConfiguration(
    $rootCatalog,
    $container,
    ServiceConfiguration::createWithDefaults()
        ->withLoadCatalog("src"),
    [],
    false
);
```

