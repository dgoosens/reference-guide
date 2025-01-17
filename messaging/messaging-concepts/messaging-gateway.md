---
description: Message Gateway PHP
---

# Messaging Gateway

![](../../.gitbook/assets/gateway\_execution.svg)

The Messaging Gateway encapsulates messaging-specific code (The code required to send or receive a [Message](message.md)) and separates it from the rest of the application code.\
It does build from domain specific objects a [Message](message.md) that is send via [Message channel.](message-channel.md) \
To not have dependency on the _Ecotone API_ — including the gateway class, _Ecotone_ provides the Gateway as interface. Framework generates a proxy for any interface and internally invokes the gateway methods. By using dependency injection, you can then expose the interface to your business methods.

### Implementing own Gateway

```php
namespace Product;

use Ecotone\Messaging\Annotation\Gateway;

//1
interface ProductGateway
{
    #[Gateway("buyProduct")]  // 2
    public function order(string $productName) : void;
}
```

1. By default gateway will be available under interface name, in that case `Product\ProductGateway.`\
   If you want to register it under different name for example "productGateway", then pass it to annotation `#[ClassReference("productGateway")]`

{% tabs %}
{% tab title="Symfony / Laravel" %}
If you are using Symfony Integration, then it will be auto registered in Dependency Container with possibility to auto-wire the gateway.

{% hint style="info" %}
As Command/Query/Event buses are Gateways, you can auto-wire them. \
They can be injected into Controller and called directly.
{% endhint %}
{% endtab %}

{% tab title="Lite" %}
In Lite Configuration you can retrieve it using messaging system configuration.

```php
$productGateway = $messagingSystem->getGatewayByName(ProductGateway::class);
```
{% endtab %}
{% endtabs %}

&#x20; 2\. `Gateway` enables method to be used as Messaging Gateway. You may have multiple Gateways defined within interface.

### Gateway reply

```php
    #[Gateway("getPrice")] 
    public function getPrice(string $productName) : int;
```

Gateway may return values, but as you probably remember, everything is connected via [Message Channels](message-channel.md). So how does we get the reply? \
During [Message](message.md) building, gateway adds header `replyChannel` which contains automatically created Channel. During [Endpoint's method invocation](../conversion/method-invocation.md), if any value was returned it will be sent via reply Channel. \
This way gateway may receive the reply and return it.&#x20;

### Parameters to Message Conversion

In order to build [Message](message.md),  Parameter Converters are introduced. \
You may configure them manually or let _Ecotone_ make use of default parameter converters.

```php
#[Gateway("orders")]
public function placeOrder(#[Payload] Order $order, #[Header("executorId")] string $executorId);
```

Parameter converter types:

* This will convert string passed under `$content` parameter to message payload

```php
public function sendMail(#[Payload] string $content) : void;
```

* This convert `$content` to message's payload and will add to headers under "_receiverEmail_" key value of `$toEmail`

```php
public function sendMail(#[Payload] string $content, #[Header()] string $toEmail) : void;
```

