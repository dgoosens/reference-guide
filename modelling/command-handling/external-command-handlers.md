# External Command Handlers

### External Command Handlers

`External Command Handlers` are handlers used as services available in your dependency container.

```php
<?php

namespace Ecotone;

use Ecotone\Modelling\Annotation\CommandHandler;

class TicketApi
{
    /**
     * @CommandHandler() //1
     */
    public function startTicket(StartTicketCommand $command) : void
    {
        // do something with buy book command
    }
}
```

1. A `@CommandHandler` annotated method are places where you would put your business logic. This annotation tells the framework that the given method is capable of handling the `StartTicketCommand`.

{% hint style="info" %}
If you are using autowire functionality, then all your classes are registered using class names.   
In other case, if your class name are not corresponding to their name in Dependency Container, then you may tell `Ecotone` about it, using `@ClassReference`.



* ```php
  @ClassReference("ticketApi")
  class TicketApi
  ```
{% endhint %}

