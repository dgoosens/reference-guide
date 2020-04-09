# External Command Handlers

### External Command Handlers

`External Command Handlers` are handlers used as services available in your dependency container.

```php
<?php

namespace Ecotone;

use Ecotone\Messaging\Annotation\MessageEndpoint;
use Ecotone\Modelling\Annotation\CommandHandler;

/**
 * @MessageEndpoint() //1
 */
class TicketApi
{
    /**
     * @CommandHandler() //2
     */
    public function startTicket(StartTicketCommand $command) : void
    {
        // do something with buy book command
    }
}
```

There are a couple of noteworthy concepts from the given code snippet, marked with numbered PHP comments referring to the following bullets:

1. Ecotone search for classes with _`@MessageEndpoint`_ annotation.   
   Thanks to it, framework can know about the class and under which name, it's registered in Depedency Container. By default class name is used, in this case it will be _`Ecotone\TicketApi`._ 

   If this class would be available under different name e.g. _"ticketApi"_, then it would be registered using: 

   ```php
   @MessageEndpoint(referenceName="ticketApi")
   ```

   If you're using autowire functionality, then all your classes are registered using class names. In that case, you don't need to add _referenceName._   

2. A `@CommandHandler` annotated method are places where you would put your business logic. This annotation tells the framework that the given method is capable of handling the `StartTicketCommand`.

{% hint style="info" %}
If you are using autowire functionality, then all your classes are registered using class names. In that case, you don't need to add _referenceName._ 
{% endhint %}

