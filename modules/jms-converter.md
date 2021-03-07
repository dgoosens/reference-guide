# JMS Converter

## Installation

{% hint style="success" %}
`composer require ecotone/jms-converter`
{% endhint %}

Ecotone comes with integration with [JMS Serializer](https://jmsyst.com/libs/serializer) and extending it with extra features.

### Module Powered By

Great library, which allow for advanced conversion between types [JMS/Serializer](https://github.com/schmittjoh/serializer).

## Conversion Table

  
`JMS Converter` can handle conversions:

```php
// conversion from JSON to PHP
application/json => application/x-php | {"productId": 1} => new OrderProduct(1)
// conversion from PHP to JSON
application-x-php => application/json | new OrderProduct(1) => {"productId": 1}

// conversion from XML to PHP
application/xml => application/x-php | <productId>1</productId> => new OrderProduct(1)
// conversion from PHP to XML
application-x-php => application/xml | new OrderProduct(1) => <productId>1</productId>

// conversion from JSON to PHP Array
application/json => application/x-php;type=array | {"productId": 1} => ["productId": 1]
// conversion from PHP Array to JSON
application/x-php;type=array => application/json | {"productId": 1} => ["productId": 1]

// conversion from XML to PHP Array
application/xml => application/x-php;type=array | <productId>1</productId> => ["productId": 1]
// conversion from PHP Array to XML
application/x-php;type=array => application/xml | ["productId": 1] => <productId>1</productId> 
```

## How to register Converter

`JMS Converter`make use of Converters registered as `PHP to PHP` Converters in order to provide all the conversion types described in [Conversion Table](jms-converter.md#conversion-table). You can read how to register new `PHP to PHP Converter` in [Conversion section.](../messaging/conversion/conversion.md#conversions-on-php-level)

## Example usage

Suppose we have endpoint with following Command:

```php
/**
*  @CommandHandler("order.place")
*/
public function placeOrder(PlaceOrder $orderId)
{
   // do something
}
```

```php
class PlaceOrder
{
    /**
     * @var Uuid[]
     */
    private array $productIds;
    
    private ?string $promotionCode;
    
    private bool $quickDelivery;
}
```

We do not need to add any metadata describing how to convert `JSON to PlaceOrder PHP class.` We already have it using type hints. 

{% hint style="info" %}
If use you PHP 7.3, you may describe types using docblocks. 
{% endhint %}

The only thing, that we need is to add how to convert string to UUID. We do it using PHP to PHP Converter:

```php
use Ecotone\Messaging\Annotation\Converter;
use Ecotone\Messaging\Annotation\ConverterClass;

/**
 * @ConverterClass()
 */
class ExampleConverterService
{
    /**
     * @Converter()
     */
    public function convert(string $data) : Uuid
    {
        return Uuid::fromString($data);
    }
}
```

And that's enough. Whenever we will use `string to UUID` conversion or `array of string to array of UUID`. This converter will be automatically used.   
  
Now we can call `Command Bus` with JSON.

```php
$this->commandBus->convertAndSend(
   "order.place", 
   "application/json", 
   '{"productIds": ["104c69ac-af3d-44d1-b2fa-3ecf6b7a3558"], "promotionCode": "33dab", "quickDelivery": false}'
)
```

## Customization

If you want to customize serialization or deserialization process, you may use of annotations on properties, just like it is describes in [Annotation section in JMS Serializer](https://jmsyst.com/libs/serializer/master/reference/annotations).

```php
class GetOrder
{
   /**
   * @SerializedName("order_id")
   */
   private string $orderId;
}
```

