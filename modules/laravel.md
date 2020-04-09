# Laravel

## Installation

{% hint style="success" %}
`composer require ecotone/laravel`
{% endhint %}

Provider should be automatically registered.

2. Register provider, if needed

```php
'providers' => [
    \Ecotone\Laravel\EcotoneProvider::class
],
```

## Configuration

```php
loadAppNamespaces: bool (default: true)
cacheConfiguration: bool (default: false, production: true)
namespaces: string[] (default: [])
serializationMediaType: string (default: application/x-php-serialized) [application/json, application/xml]
defaultErrorChannel: string (default: null)
```

### loadAppNamespaces

Tells Ecotone, if should automatically load all namespaces defined for `app` catalog

### cacheConfiguration

Describes if Ecotone should cache configuration.   
If `true`, then Ecotone will cache all configurations this will increase application load time, but will results in slower feedback for the developer as cache will need to be removed on change  
if `false,` then Ecotone will not cache configuration this will decrease application load time, but will results in quicker feedback for the developer

### namespaces

List of namespace prefixes, that should be loaded aby Ecotone 

### serializationMediaType

Describes default serialization type within application. If not configured default serialization will be `application/x-php-serialized,` ~~~~which is sierliazed PHP class.

### defaultErrorChannel

Provides default [Poller configuration](../messaging/asynchronous.md#polling-metadata) with error channel for all pollable endpoints.

