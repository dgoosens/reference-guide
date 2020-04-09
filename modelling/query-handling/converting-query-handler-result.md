# Converting Query Handler Result

If you have registered [Converter](../../messaging/conversion/) for specific Media Type, then you can tell `Ecotone` to convert result of any [Gateway](../../messaging/messaging-concepts/messaging-gateway.md) to specific format. This is especially useful, when we are dealing with `QueryBus`, when we want to return the result to the caller of the request.   
In order to do this, we need to make use of `Metadata`and `replyContentType` header.

{% tabs %}
{% tab title="Symfony" %}
```php
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
      return new Response(
         $this->queryBus->sendWithMetadata(
            new GetTicketStatusQuery($request->get("ticketId")),
            ["replyContentType" => "application/json"]
         );
      )    
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
use Ecotone\Modelling\QueryBus;

$queryBus = $messagingSystem->getGatewayByName(QueryBus::class);

// result will be in json
$result = $queryBus->sendWithMetadata(
             new GetTicketStatusQuery($ticketId), 
             ["replyContentType" => "application/json"]
          );
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Handler" %}
```php
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\QueryHandler;

/**
 *  @MessageEndpoint()
 */
class GetTicketStatusQueryHandler
{   
    /**
    * @QueryHandler()
    */
    public function getTicketStatus(GetTicketStatusQuery $query)
    {
        return ["ticketId" => $query->getTicketId(), "status" => "inProgress"];
    }   
}
```
{% endtab %}
{% endtabs %}

