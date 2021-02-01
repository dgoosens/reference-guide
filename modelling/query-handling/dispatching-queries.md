# Dispatching Queries

Previous pages provide the background on how to handle query messages in your application. The dispatching process is the starting point for query message.

### Query Bus

`Query Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface QueryBus
{
    1 public function send(object $query);

    2 public function sendWithMetadata(object $query, array $metadata);

    3 public function convertAndSend(string $name, string $sourceMediaType, $queryData);

    4 public function convertAndSendWithMetadata(string $name, string $sourceMediaType, $queryData, array $metadata);
}
```

### Send method

Query is routed to the Handler by class type.

{% tabs %}
{% tab title="Symfony" %}
```php
// Query Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\QueryBus;

class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request) : Response
   {
      $result = $this->queryBus->send(
         new GetTicketStatusQuery($request->get("ticketId"))
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
use Ecotone\Modelling\QueryBus;

$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

$result = $queryBus->send(new GetTicketStatusQuery($ticketId));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
use Ecotone\Modelling\Annotation\QueryHandler;

class GetTicketStatusQueryHandler
{   
    /**
    * @QueryHandler()
    */
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

### SendWithMetadata

Does allow for passing `extra meta information`, that can be used on targeted `Query Handler`.

{% tabs %}
{% tab title="Symfony" %}
```php
// Query Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Security;
use Ecotone\Modelling\QueryBus;

class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request, Security $security) : Response
   {
      $result = $this->queryBus->sendWithMetadata(
         new GetTicketStatusQuery($request->get("ticketId")),
         ["executorUsername" => $security->getUser()->getUsername()]
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
use Ecotone\Modelling\QueryBus;

$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

$result = $queryBus->sendWithMetadata(
   new GetTicketStatusQuery($ticketId),
   ["executorUsername" => $executorUsername]
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
use Ecotone\Modelling\Annotation\QueryHandler;

class GetTicketStatusQueryHandler
{   
    /**
    * @QueryHandler()
    */
    public function getTicketStatus(GetTicketStatusQuery $query, array $metadata)
    {
          $ticket = ; // get Ticket using query
    
          if ($metadata["executorUsername"] !== $ticket->getOwner()) {
             throw new \InvalidArgumentException("Insufficient permissions")
          }
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

### ConvertAndSend

Is used with `Query Handlers,` routed by name and converted using [Converter](../../messaging/conversion/) if needed.

{% tabs %}
{% tab title="Symfony" %}
```php
// Query Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\QueryBus;

class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request) : Response
   {
      $result = $queryBus->convertAndSend("getTicketStatus", "application/json", '{"ticketId": 123}');
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
use Ecotone\Modelling\QueryBus;

$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

$result = $queryBus->convertAndSend(
   "getTicketStatus", 
   "application/json", 
   '{"ticketId": 123}'
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
use Ecotone\Modelling\Annotation\QueryHandler;

class GetTicketStatusQueryHandler
{   
    /**
    * @QueryHandler(inputChannelName="getTicketStatus")
    */
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

### ConvertAndSendWithMetadata

{% tabs %}
{% tab title="Symfony" %}
```php
// Query Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\QueryBus;

class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request) : Response
   {
      $result = $queryBus->convertAndSendWithMetadata(
         "getTicketStatus", 
         "application/json", 
         '{"ticketId": 123}', 
         ["executorUsername" => $executorUsername]
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
use Ecotone\Modelling\QueryBus;

$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

$result = $queryBus->convertAndSendWithMetadata(
    "getTicketStatus", 
    "application/json", 
    '{"ticketId": 123}',
    ["executorUsername" => "someUsername"]
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
use Ecotone\Modelling\Annotation\QueryHandler;

class GetTicketStatusQueryHandler
{   
    /**
    * @QueryHandler(inputChannelName="getTicketStatus")
    */
    public function getTicketStatus(GetTicketStatusQuery $query, array $metadata)
    {
          $ticket = ; // get Ticket using query
    
          if ($metadata["executorUsername"] !== $ticket->getOwner()) {
             throw new \InvalidArgumentException("Insufficient permissions")
          }
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

