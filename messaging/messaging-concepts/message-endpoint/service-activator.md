# Service Activator

The Service Activator connecting any service available in Depedency Container to an input channel so that it may play the role of a [Endpoint](./). If the service produces output, it may also be connected to an output channel.   
Alternatively, an output producing service may be located at the end of a processing pipeline or message flow in which case, the inbound Message's "replyChannel" header can be used. This is the default behavior if no output channel is defined.

### How to register

```php
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Messaging\Annotation\ServiceActivator;

/**
 * @MessageEndpoint()
 */
class Shop
{
    /**
     * @ServiceActivator(inputChannelName="buyProduct")
     */
    public function buyProduct(int $productId) : void
    {
        echo "Product with id {$productId} was bought";
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

