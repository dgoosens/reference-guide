# Dispatching Commands

Previous pages provide the background on how to handle command messages in your application. The dispatching process is the starting point for command message.

### Command Bus

`Command Bus` is special type of Messaging Gateway. 

```php
namespace Ecotone\Modelling;

interface CommandBus
{
    1 public function send(object $command);

    2 public function sendWithMetadata(object $command, array $metadata);

    3 public function convertAndSend(string $name, string $sourceMediaType, $commandData);

    4 public function convertAndSendWithMetadata(string $name, string $sourceMediaType, $commandData, array $metadata);
}
```

### Send method

Command is routed to the Handler by class type.

{% tabs %}
{% tab title="Symfony" %}
```php
// Command Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\CommandBus;

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
use Ecotone\Modelling\CommandBus;

$commandBus = $messagingSystem->getGatewayByName(CommandBus::class);

$commandBus->send(new CloseTicketCommand($ticketId));
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;

/**
 *  @MessageEndpoint()
 */
class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
    public function closeTicket(CloseTicketCommand $command)
    {
//        handle closing ticket
    }   
}
```
{% endtab %}
{% endtabs %}

### SendWithMetadata

Does allow for passing `extra meta information`, that can be used on targeted `Command Handler`.

{% tabs %}
{% tab title="Symfony" %}
```php
// Command Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Security\Core\Security;
use Ecotone\Modelling\CommandBus;

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
use Ecotone\Modelling\CommandBus;

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
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;

/**
 *  @MessageEndpoint()
 */
class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
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

### ConvertAndSend

Is used with `Command Handlers,` routed by name and converted using [Converter](../../messaging/conversion/) if needed.

{% tabs %}
{% tab title="Symfony" %}
```php
// Command Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\CommandBus;

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
use Ecotone\Modelling\CommandBus;

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
use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;

/**
 *  @MessageEndpoint()
 */
class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler(inputChannelName="closeTicket")
    */
    public function closeTicket(CloseTicketCommand $command)
    {
//        handle closing ticket
    }   
}
```
{% endtab %}
{% endtabs %}

### ConvertAndSendWithMetadata

{% tabs %}
{% tab title="Symfony" %}
```php
// Command Bus will be auto registered in Depedency Container.

use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\HttpFoundation\Request;
use Ecotone\Modelling\CommandBus;

class TicketController
{
   private CommandBus $commandBus;

   public function __construct(CommandBus $commandBus)
   {
       $this->commandBus = $commandBus;   
   }
   
   public function closeTicketAction(Request $request) : Response
   {
      $commandBus->convertAndSendWithMetadata(
         "closeTicket", 
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
use Ecotone\Modelling\CommandBus;

$commandBus = $messagingSystem->getGatewayByName(CommandBus::class);

$commandBus->convertAndSendWithMetadata(
   "closeTicket", 
   "application/json", 
   '{"ticketId": 123}',
   ["executorUsername" => "someUsername"]
);
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Command Handler" %}
```php
use Ecotone\Modelling\Annotation\CommandHandler;

/**
 *  @MessageEndpoint()
 */
class CloseTicketCommandHandler
{   
    /**
    * @CommandHandler()
    */
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

