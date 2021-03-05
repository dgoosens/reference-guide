# Message Router

Routers consume messages from a message channel and forward each consumed message to one or more different message channels depending on a defined conditions.

Router must return name of the channel, where the message should be routed too. It can be array of channel names, if there are more. 

```php
class OrderRouter
{
    #[Router("order")] 
    public function orderSpecificType(string $orderType) : string
    {
        return $orderType === 'coffee' ? "orderInCoffeeShop" : "orderInGeneralShop";
    }
}
```

### Possible options

* `endpointId` - Endpoint identifier 
* `inputChannnelName` - Required option, defines to which channel endpoint should be connected
* `isResolutionRequired` - If true, will throw exception if there was no channel name returned   

