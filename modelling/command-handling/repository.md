---
description: Repository PHP
---

# Repository

Repositories are used for retrieving and saving the aggregate to persistent storage.&#x20;

Aggregate need to be loaded in order to call method on it. \
Normally the flow for calling aggregate method, would looks like below. Which involves having handling service with access to repository.

```php
class AssignWorkerCommmand
{
    private string $ticketId;
    
    private string $workerId;
    
    public function getTicketId() : string;
    {
       return $this->ticketId;
    }
    
    public function getWorkerId() : string
    {
       return $this->workerId;
    }
}


class CloseTicketHandler
{
    private TicketRepository $ticketRepository;

    #[CommandHandler]
    public function handle(CreateTicketCommand $command) : void
    {
       $ticket = $this->ticketRepository->findBy($command->getTicketId());
       $ticket->assignWorker($command->getWorkerId());
       $this->ticketRepository->save($ticket);    
    }
}

class Ticket
{
    public function assignWorker(string $workerId)
    {
       // do something with assignation
    }
}
```

_Ecotone_ provides possibility to mark Ticket Aggregate [methods as `CommandHandler` directly.](state-stored-aggregate.md) \
In that situation, Ecotone retrievies identifiers from Command message, pass them to `Repository`, calls the method on aggregate instance and saves it. In short it does code from `CreateTicketCommand Handler` for you.&#x20;

{% hint style="info" %}
As Ecotone does not try to impose specific solutions, you are free to choose, which fits you best in specific context. Above example is based on [External Command Handlers](external-command-handlers.md).
{% endhint %}

To get more details how to implement _Aggregate,_ go to previous pages:

{% content-ref url="state-stored-aggregate.md" %}
[state-stored-aggregate.md](state-stored-aggregate.md)
{% endcontent-ref %}

{% content-ref url="broken-reference" %}
[Broken link](broken-reference)
{% endcontent-ref %}

### How to implement Repository

There are two types of repositories. One for storing [`State-Stored Aggregate`](state-stored-aggregate.md) and another one for storing [`Event Sourcing Aggregate`](broken-reference).

Based on which interface is implemented, `Ecotone` knows which Aggregate type was selected.\
The interface informs, if specific `Repository` can handle given `Aggregate class.`\
You may implement&#x20;

#### Repository for State-Stored Aggregate

```php
namespace Ecotone\Modelling;

interface StandardRepository
{
    
    1 public function canHandle(string $aggregateClassName): bool; 
    
    2 public function findBy(string $aggregateClassName, array $identifiers) : ?object;
    
    3 public function save(array $identifiers, object $aggregate, array $metadata, ?int $expectedVersion): void;
}
```

1. `canHandle method` informs, which `Aggregate Classes` can be handled with this `Repository`. Return true, if saving specific aggregate is possible, false otherwise.
2. `findBy method` returns if found, existing `Aggregate instance`, otherwise null.&#x20;
3. `save method` is reponsible for storing given `Aggregate instance`.&#x20;



* `$identifiers` are array of `@AggregateIdentifier` defined within aggregate.
* `$aggregate` is instance of aggregate
* `$metadata` is array of extra information, that can be passed with Command
* `$expectedVersion` if version locking by `@Version` is enabled it will carry currently expected version

#### Repository for Event Sourced Aggregate

```php
namespace Ecotone\Modelling;

interface EventSourcedRepository
{
    public function canHandle(string $aggregateClassName): bool;
    
    1 public function findBy(string $aggregateClassName, array $identifiers) :  EventStream;

    2 public function save(array $identifiers, string $aggregateClassName, array $events, array $metadata, int $versionBeforeHandling): void;
}
```

Event Sourced Repository  instead of working with aggregate instance, works with events.&#x20;

1. `findBy method` returns previously created events for given aggregate.&#x20;
2. `save method` gets array of events to save returned by `CommandHandler` after performing an action
