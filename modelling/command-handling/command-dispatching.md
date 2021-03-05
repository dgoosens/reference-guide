# Dispatching Commands

Previous pages provide the background on how to handle command messages in your application. The dispatching process is the starting point for command message.

### Command Bus

`Command Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface CommandBus
{
    // 1
    public function send(object $command, array $metadata = []) : mixed;

    // 2
    public function sendWithRouting(string $routingKey, mixed $command, string $commandMediaType = MediaType::APPLICATION_X_PHP, array $metadata = []) : mixed;
}
```

## Send method

Command is routed to the Handler by class type.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
// Command Bus will be auto registered in Depedency Container.
class TicketController
{
   private CommandBus $commandBus;

   public function __construct(CommandBus $commandBus)
   {
       $this->commandBus = $commandBus;   
   }
   
   public function closeTicketAction(Request $request) : Response
   {
      $this->commandBus->send(
         new CloseTicketCommand($request->get("ticketId"))
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
$commandBus = $messagingSystem->getGatewayByName(CommandBus::class);

$commandBus->send(new CloseTicketCommand($ticketId));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command Handler" %}
```php
class CloseTicketCommandHandler
{   
    #[CommandHandler]
    public function closeTicket(CloseTicketCommand $command)
    {
//        handle closing ticket
    }   
}
```
{% endtab %}
{% endtabs %}

## Send With Metadata

Does allow for passing `extra meta information`, that can be used on targeted `Command Handler`.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
class TicketController
{
   private CommandBus $commandBus;

   public function __construct(CommandBus $commandBus)
   {
       $this->commandBus = $commandBus;   
   }
   
   public function closeTicketAction(Request $request, Security $security) : Response
   {
      $this->commandBus->sendWithMetadata(
         new CloseTicketCommand($request->get("ticketId")),
         ["executorUsername" => $security->getUser()->getUsername()]
      );
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
$commandBus = $messagingSystem->getGatewayByName(CommandBus::class);

$commandBus->sendWithMetadata(
   new CloseTicketCommand($ticketId),
   ["executorUsername" => $executorUsername]
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command Handler" %}
```php
class CloseTicketCommandHandler
{   
    #[CommandHandler]
    public function closeTicket(CloseTicketCommand $command, array $metadata)
    {
          $ticket = ; // get Ticket using command
    
          if ($metadata["executorUsername"] !== $ticket->getOwner()) {
             throw new \InvalidArgumentException("Insufficient permissions")
          }
//        handle closing ticket
    }   
}
```
{% endtab %}
{% endtabs %}

## Send With Routing

Is used with `Command Handlers,` routed by name and converted using [Converter](../../messaging/conversion/) if needed.

{% tabs %}
{% tab title="Symfony / Laravel" %}
```php
class TicketController
{
   private CommandBus $commandBus;

   public function __construct(CommandBus $commandBus)
   {
       $this->commandBus = $commandBus;   
   }
   
   public function closeTicketAction(Request $request) : Response
   {
      $commandBus->convertAndSend("closeTicket", "application/json", '{"ticketId": 123}');
   }
}
```
{% endtab %}

{% tab title="Lite" %}
```php
$commandBus = $messagingSystem->getGatewayByName(CommandBus::class);

$commandBus->convertAndSend(
   "closeTicket", 
   "application/json", 
   '{"ticketId": 123}'
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command Handler" %}
```php
class CloseTicketCommandHandler
{   
    #[CommandHandler]
    public function closeTicket(CloseTicketCommand $command)
    {
//        handle closing ticket
    }   
}
```
{% endtab %}
{% endtabs %}

