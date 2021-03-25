---
description: Conversion PHP
---

# Conversion

Command, queries and events are not always objects. When they travel via different channels, coming from outside, they are mostly converted to simplified format, like `JSON` or `XML`.    
At the level of application however we want to deal with it in `PHP` format, as objects or arrays , so we can understand what are we dealing with.

Moving from one format to another requires conversion. `Ecotone` does it automatically using [Media Type](https://pl.wikipedia.org/wiki/Typ_MIME) Converters.   
`Media Type Converters` are responisble for converting data into expected format. It can be from `JSON to PHP`, but it also can be other way around from `PHP to JSON`.

## Media Type Converter

We need to define class for it which implements `Converter` and is marked by annotation `@MediaTypeConverter.`  

```php
#[MediaTypeConverter] 
class CustomConverter implements Converter
{
    public function matches(TypeDescriptor $sourceType, MediaType $sourceMediaType, TypeDescriptor $targetType, MediaType $targetMediaType): bool
    {

    }

    public function convert($source, TypeDescriptor $sourceType, MediaType $sourceMediaType, TypeDescriptor $targetType, MediaType $targetMediaType)
    {

    }
}
```

There are two methods `matches` and `convert.`   
`Matches` tells Ecotone, if specific Converter can do conversion, by returning `true/false`.  
`Convert` does actual conversion from one type to another. 

1. `TypeDescriptor` - Describes expected type in PHP format. This can be `class, scalar type, array` etc. 
2. `MediaType` - Describes expected Media Type format. This can be `application/json`, `application/xml` or `application/x-php` etc. 
3. `$source` - is the actual data to be converted. 

### How does it apply to actual endpoint execution

Suppose we have `Command Handler` endpoint, which expects `PlaceOrderCommand` class.

```php
#[CommandHandler]
public function placeOrder(PlaceOrderCommand $command)
{
   // do something
}
```

In our HTTP Controller, we receive `JSON` and we want to send it to Command Bus:

```php
$this->commandBus->sendWithRouting(
   "order.place", 
   '{"productIds": [1,2]}',
   "application/json"
)
```

`Ecotone`does delay the conversion to the time, when it's actually needed. In that case it will be just before `placeOrder` method will be called.   
Then right before execution `Ecotone` will resolve, that Payload of [Message](../messaging-concepts/message.md), which will be `{"productIds": [1,2]}` and it's content Type `application/json` differs from the expected type, which is `PlaceOrderCommand` and content Type `application/x-php` \(default for PHP types\). 

It's converter that `matches` JSON to PHP conversion will be used to do the conversion:

```php
    public function convert($source, TypeDescriptor $sourceType, MediaType $sourceMediaType, TypeDescriptor $targetType, MediaType $targetMediaType)
    {
       // $source - {"productIds": [1,2]}
       // $sourceType - string
       // $sourceMediaType - application/json
       // $targetType - PlaceOrderCommand
       // $targetMediaType - application/x-php
    }
```

## How to build your own Media Type Converter

If we would like to implement `JSON to PHP` converter, we could start like this:

```php
public function matches(TypeDescriptor $sourceType, MediaType $sourceMediaType, TypeDescriptor $targetType, MediaType $targetMediaType): bool
{
    return $sourceMediaType->isCompatibleWith(MediaType::createApplicationJson()) // if source media type is JSON
        && $targetMediaType->isCompatibleWith(MediaType::createApplicationXPHPObject())    ; // and target media type is PHP
}
```

This will tell `Ecotone` that in case source media type is `JSON` and target media type is `PHP`, then it should use this converter.   
Then you could inject into the class specific converter of your use for example `Symfony Serializer` and make use of it in convert method.

```php
public function convert($source, TypeDescriptor $sourceType, MediaType $sourceMediaType, TypeDescriptor $targetType, MediaType $targetMediaType)
{
    return $this->serializer->deserialize($source, $targetType->toString(), "json");
}
```

## Conversions on PHP Level

`Ecotone` does come with simplication for PHP level conversion. Suppose we want to send Query as scalar type. 

```php
$this->queryBus->sendWithRouting(
   "order.getDetails", 
   "61cfc7ea-928f-420f-a8e1-656ae2968254",
   MediaType::APPLICATION_X_PHP
)
```

```php
#[QueryHandler("order.getDetails")]
public function getOrderDetails(Uuid $orderId)
{
   // do something
}
```

{% hint style="info" %}
As you can see `query` neither, `command` needs to be class. It can be simple array or even a scalar, it's really up to the developer, what does fit best for him in given scenario. 
{% endhint %}

Let's add PHP Conversion:e

```php
class ExampleConverterService
{
    #[Converter] 
    public function convert(string $data) : Uuid
    {
        return Uuid::fromString($data);
    }
}
```

`Ecotone`will read parameter type, which is `string` and return type, which is `Uuid.`   
Based on that fact, converter from `string` to `Uuid` will be registered and our query handler will be called with success.  
  
Conversion does work with more complicated objects, expecting more than one parameter.  
In that case array can be used in order to construct the object.

```php
class ExampleConverterService
{
    #[Converter] 
    public function convert(array $data) : Coordinates
    {
        return Coordinates::create($data["latitude"], $data["longitude"]);
    }
}
```

### Conversion Array of objects

Suppose we expect array of UUID's. 

```php
$this->queryBus->sendWithRouting(
   "order.getDetails", 
   ["61cfc7ea-928f-420f-a8e1-656ae2968254", "d32ee0cc-fd01-474d-b69f-2d2489433f3d"],
   MediaType::APPLICATION_X_PHP
)
```

```php
/**
*  @param Uuid[] $orderIds
*/
#[QueryHandler("order.getDetails")]
public function getOrders(array $orderIds)
{
   // do something
}
```

In order to handle such conversion, we do not need to do anything more. We have converter for `string to UUID` then it will be automatically used in order to handle array of string for UUID conversion.  
  
`Ecotone` does know that parameter `array $orderIds`is array of UUIDs based on docblock parameter  
`@param Uuid[] $orderIds.` 

## Serializer

You may want to serialize/deserialize on the level of your application code using Converters you have already registered. In order to do that `Ecotone` comes with `Serializer` [`Gateway`](../messaging-concepts/messaging-gateway.md)

Serializer Gateway is built from simple interface

```php
namespace Ecotone\Messaging\Gateway\Converter;

interface Serializer
{
    public function convertFromPHP($data, string $targetMediaType);

    public function convertToPHP($data, string $sourceMediaType, string $targetType);
}
```

`convertFromPHP` - This method is responsible for converting source PHP `$data` to specific `target Media Type.` 

```php
$this->serializer->convertFromPHP([1,2,3], "application/json")
```

  
`convertToPHP` - This method is responsible for converting source with specific `media type` to target `PHP type.`

```php
$this->serializer->convertToPHP('{"productId": 1}', "application/json", OrderProduct:class)
```

### Injecting Serializer

As Serializer is `Gateway` it will be automatically registered in your Dependency Container, under class name.

