# Splitter

The Splitter is endpoint where message can be splitted in several parts and be sent to be processed indepedently. 

```php
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Messaging\Annotation\Splitter;
use Ecotone\Messaging\Annotation\ServiceActivator;

/**
 * @MessageEndpoint()
 */
class Shop
{
    /**
     * @Splitter(inputChannelName="buyProduct", outputChannelName="buySingleProduct")
     */
    public function sendMultipleOrders(array $products) : array
    {
        return $products;
    }

    /**
     * @ServiceActivator(inputChannelName="buySingleProduct")
     */
    public function buyProduct(string $productName) : void
    {
        echo "Product {$productName} was bought";
    }
}
```

### Possible options

* `endpointId` - Endpoint identifier 
* `inputChannnelName` - Required option, defines to which channel endpoint should be connected
* `parameterConverters` - Manual configured [parameter converters]()
* `poller` - [Pollable Metadata]() configuration for pollable input channel
* `outputChannelName` - Channel where result of method invocation will be 
* `requiredInterceptorNames` - List of [interceptor](../../interceptors.md) names, which should intercept the endpoint

