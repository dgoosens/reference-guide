---
description: Event Sourcing DDD CQRS Symfony Laravel
---

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
defaultMemoryLimit: string (default: null)
defaultChannelPollRetry: 
   initialDelay: int (default: 100, production: 1000)
   maxAttempts: int (default: 3, production: 5)
   multiplier: int (default: 3)
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

Describes default serialization type within application. If not configured default serialization will be `application/x-php-serialized,`which is serialized PHP class.

### defaultErrorChannel

Provides default [Poller configuration](../messaging/scheduling.md#polling-metadata) with error channel for all asynchronous endpoints.

### defaultMemoryLimit

Provides default memory limit in megabytes for all asynchronous endpoints

### defaultChannelPollRetry

Provides default channel poll retry template. It's responsible for connection retry for asynchronous endpoints.

`initialDelay` - delay after first retry in milliseconds  
`multiplier` - how much initialDelay should be multipled with each try  
`maxAttempts` - How many attemps should be done, before closing closing endpoint

