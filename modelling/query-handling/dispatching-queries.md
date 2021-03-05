# Dispatching Queries

Previous pages provide the background on how to handle query messages in your application. The dispatching process is the starting point for query message.

### Query Bus

`Query Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface QueryBus
{
    public function send(object $query, array $metadata = []) : mixed;

    public function sendWithRouting(string $routingKey, mixed $query, string $queryMediaType = MediaType::APPLICATION_X_PHP, array $metadata = []) : mixed;
}
```

## Send method

Query is routed to the Handler by class type.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
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
$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

$result = $queryBus->send(new GetTicketStatusQuery($ticketId));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
class GetTicketStatusQueryHandler
{   
    #[QueryHandler]
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

## Sending With Metadata

Does allow for passing `extra meta information`, that can be used on targeted `Query Handler`.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
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
class GetTicketStatusQueryHandler
{   
    #[QueryHandler]
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

## Send With Routing

Is used with `Query Handlers`routed by name and converted using [Converter](../../messaging/conversion/) if needed.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
class TicketController
{
   private QueryBus $queryBus;

   public function __construct(QueryBus $queryBus)
   {
       $this->queryBus = $queryBus;   
   }
   
   public function getTicketStatusAction(Request $request) : Response
   {
      $result = $queryBus->convertAndSend(
         "getTicketStatus", 
         "application/json", 
         '{"ticketId": 123}'
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
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
class GetTicketStatusQueryHandler
{   
    #[QueryHandler("getTicketStatus")]
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
//        handle retrieving ticket status
    }   
}
```
{% endtab %}
{% endtabs %}

