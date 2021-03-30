# Ecotone Lite

## Installation

Follow Installation from [Installation menu](../install-php-service-bus.md#install-lite-no-framework).

## Configuration

```php
$messagingSystem = EcotoneLiteConfiguration::createWithConfiguration(
    $rootCatalog,
    $container,
    ServiceConfiguration::createWithDefaults()
        ->withEnvironment("prod")
        ->withLoadCatalog("src")
        ->withFailFast(false)
        ->withNamespaces(["App"])
        ->withDefaultSerializationMediaType("application/json")                
        ->withDefaultErrorChannel("errorChannel")
        ->withConsumerMemoryLimit(512)
        ->withCacheDirectoryPath("/var/www/cache")
        ->withConnectionRetryTemplate(RetryTemplateBuilder::fixedBackOff(100))
    [],
    $useCachedVersion
);
    
```

`$rootCatalog` - Path to root catalog of the project  
`$container` - PSR compatible container   
`$useCachedVersion` - Should Ecotone use cached version of Ecotone Lite, if available

## ServiceConfiguration

### withEnvironment

Tells Ecotone what kind of environment type is currently running

### withLoadCatalog

Tells Ecotone, if to automatically load all namespaces defined for given catalog.

### withFailFast

Describes if Ecotone should fail fast.   
If `true`, then Ecotone will boot all endpoints during each request, so it can inform, if configuration is incorrect immediately, it provides fast feedback for the developer.  
if `false,` then Ecotone will not boot up any endpoints at each request, which will increase performance, but will results in slower feedback for the developer.

### withNamespaces

List of namespace prefixes, that should be loaded aby Ecotone 

### withDefaultSerializationMediaType

Describes default serialization type within application. If not configured default serialization will be `application/x-php-serialized,`which is serialized PHP class.

### withDefaultErrorChannel

Provides default [Poller configuration](../modelling/scheduling.md#polling-metadata) with error channel for all [asynchronous consumers](../messaging/messaging-concepts/consumer.md#polling-consumer).

### withDefaultMemoryLimit

Provides default memory limit in megabytes for all [asynchronous consumers](../messaging/messaging-concepts/consumer.md#polling-consumer).

### withDefaultConnectionExceptionRetry

Provides default connection retry strategy for [asynchronous consumers](../messaging/messaging-concepts/consumer.md#polling-consumer) in case of connection failure. 

`initialDelay` - delay after first retry in milliseconds  
`multiplier` - how much initialDelay should be multipled with each try  
`maxAttempts` - How many attemps should be done, before closing closing endpoint

