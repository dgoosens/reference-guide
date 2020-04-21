# Symfony Bundle

## Installation

{% hint style="success" %}
`composer require ecotone/symfony-bundle`
{% endhint %}

If you're using _`Symfony Flex`_,  bundle will auto-configure. 

2. Register bundle, if needed

{% hint style="success" %}
new Ecotone\SymfonyBundle\EcotoneSymfonyBundle::class =&gt; \['all' =&gt; true\]
{% endhint %}

## Configuration

```php
ecotone:
    loadSrcNamespaces: bool (default: true)
    failFast: bool (default: true, production: false)
    namespaces: string[] (default: [])
    serializationMediaType: string (default: application/x-php-serialized) [application/json, application/xml]
    defaultErrorChannel: string (default: null)
    defaultMemoryLimit: string (default: 1024)
    defaultChannelPollRetry: 
       initialDelay: int (default: 100, production: 1000)
       maxAttempts: int (default: 3, production: 5)
       multiplier: int (default: 3)
    
```

### loadSrcNamespaces

Tells Ecotone, if should automatically load all namespaces defined for `src` catalog

### failFast

Describes if Ecotone should fail fast.   
If `true`, then Ecotone will boot all endpoints during each request, so it can inform, if configuration is incorrect immediately, it provides fast feedback for the developer.  
if `false,` then Ecotone will not boot up any endpoints at each request, which will increase performance, but will results in slower feedback for the developer.

### namespaces

List of namespace prefixes, that should be loaded aby Ecotone 

### serializationMediaType

Describes default serialization type within application. If not configured default serialization will be `application/x-php-serialized,`which is serialized PHP class.

### defaultErrorChannel

Provides default [Poller configuration](../messaging/asynchronous.md#polling-metadata) with error channel for all asynchronous endpoints.

### defaultMemoryLimit

Provides default memory limit in megabytes for all asynchronous endpoints

### defaultChannelPollRetry

Provides default channel poll retry template. It's responsible for connection retry for asynchronous endpoints.

`initialDelay` - delay after first retry in milliseconds  
`multiplier` - how much initialDelay should be multipled with each try  
`maxAttempts` - How many attemps should be done, before closing closing endpoint

