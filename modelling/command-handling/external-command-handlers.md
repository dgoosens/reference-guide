---
description: Commands PHP
---

# External Command Handlers

### External Command Handlers

`External Command Handlers` are handlers used as services available in your dependency container.

```php
<?php

namespace Ecotone;

use Ecotone\Modelling\Attribute\CommandHandler;

class TicketApi
{
    #[CommandHandler] 
    public function startTicket(StartTicketCommand $command) : void
    {
        // do something with buy book command
    }
}
```

`#[CommandHandler]` annotated method are places where you would put your business logic.  
This annotation tells the framework that the given method is capable of handling the `StartTicketCommand`.

{% hint style="info" %}
How to publish commands, you may see in [Dispatching Command section](command-dispatching.md)
{% endhint %}

{% hint style="info" %}
If you are using autowire functionality, then all your classes are registered using class names.   
In other case, if your class name is not corresponding to their name in Dependency Container, then you may tell `Ecotone` about it, using `ClassReference`.



* ```php
  #[ClassReference("ticketApi")]
  class TicketApi
  ```
{% endhint %}

