# Command, event and query buses

* [Command bus](#command_bus)
* [Event bus](#event_bus)
* [Query bus](#query_bus)
* [Registering handlers](#registering_handlers)

--------------------------------------------------------

The bus library includes command, event and query bus implementations. The three buses are very similar but they serve different purposes so let's have a look at the differences.

###### Command bus:

The command bus is used to handle an action. One example is a `CreateUserCommand` command which creates a new user. Commands should be handled by a single command handler and the handler must not return any values.

###### Event bus:

The event bus is used to handle an event. One example is a `UserCreatedEvent` event. Events can be handled by zero to an infinite number of event handlers and the handlers must not return any values.

###### Query bus:

The query bus is used to handle an event. One example is a `GetUserQuery` query. Queries should be handled by a single query handler and the handler should return data. Note that queries should *not* alter the state of the application.

> Commands, events and queries should be simple DTO objects while handlers can be any type of [callable](https://www.php.net/manual/en/language.types.callable.php). We suggest using invokable classes as handlers as they allow for dependencies to be injected through the constructor thanks to the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection).

> Command, event and query bus instances can be injected into your CLI commands and HTTP controllers using the `CommandBusInterface`, `EventBusInterface` and `QueryBusInterface` type hints.

--------------------------------------------------------

<a id="command_bus"></a>

### Command bus

First let's create a simple DTO class for holding the command data.

```
readonly class CreateUserCommand
{
    public function __construct(
        public string $email,
        public string $username,
        public string $password
    )
    {}
}
```

Next we'll create a handler using an invokable class.

```
class CreateUserHandler
{
    public function __construct(
		protected Gatekeeper $gatekeeper
	)
	{}

    public function __invoke(CreateUserCommand $command): void
    {
        $email    = $command->email;
		$username = $command->username;
		$password = $command->password;

		$this->gatekeeper->createUser($email, $username, $password);
    }
}
```

For the sake of simplicity we'll register the handler by calling the `CommandBus::registerHandler()` method. Head down to the "[registering handlers](#registering_handlers)" section of the documentation to see the suggested way of registering handlers in your application.

```
$this->commandBus->registerHandler(CreateUserCommand::class, CreateUserHandler::class);
```

Finally we can handle our command using the command bus.

```
$this->commandBus->handle(new CreateUserCommand('foo@example.org', 'foo', 'supersecurepassword'));
```

--------------------------------------------------------

<a id="event_bus"></a>

### Event bus

First let's create a simple DTO class for holding the event data.

```
readonly class UserCreatedEvent
{
    public function __construct(
        public int $email,
        public string $username,
    )
    {}
}
```

Next we'll create a handler using an invokable class.

```
class UserCreatedHandler
{
    public function __construct(
		protected Mailer $mailer
	)
	{}

    public function __invoke(UserCreatedEvent $event): void
    {
        $email    = $event->email;
        $username = $event->username;

        $this->mailer->sendWelcomeMail($email, $username);  
    }
}
```

For the sake of simplicity we'll register the handler by calling the `EventBus::registerHandler()` method. Head down to the "[registering handlers](#registering_handlers)" section of the documentation to see the suggested way of registering handlers in your application.

```
$this->eventBus->registerHandler(UserCreatedEvent::class, UserCreatedHandler::class);
```

Finally we can handle our event using the event bus.

```
$this->eventBus->handle(new UserCreatedEvent('foo@example.org', 'foo'));
```

--------------------------------------------------------

<a id="query_bus"></a>

### Query bus

First let's create a simple DTO class for holding the query data.

```
readonly class GetUserQuery
{
    public function __construct(
        public int $id
    )
    {}
}
```

Next we'll create a handler using an invokable class.

```
class GetUserHandler
{
    public function __construct(
		protected ConnectionManager $db
	)
	{}

    public function __invoke(GetUserQuery $query): ?object
    {
        $id = $query->id;

        return $this->db->getConnection()->getQuery()
        ->from('users')
        ->where('id', '=', $id)
        ->first();
    }
}
```

For the sake of simplicity we'll register the handler by calling the `QueryBus::registerHandler()` method. Head down to the "[registering handlers](#registering_handlers)" section of the documentation to see the suggested way of registering handlers in your application.

```
$this->queryBus->registerHandler(GetUserQuery::class, GetUserHandler::class);
```

Finally we can handle our query using the query bus.

```
$user = $this->queryBus->handle(new GetUserQuery(1));
```

--------------------------------------------------------

<a id="registering_handlers"></a>

### Registering handlers

The Mako skeleton application comes with a `BusService` that allows you to register your command, event and query handlers in a sentralized place using the `getCommandHandlers`, `getEventHandlers` and `getQueryHandlers` methods.

```
<?php

namespace app\services;

use app\commands\CreateUserCommand;
use app\commands\CreateUserHandler;
use app\events\UserCreatedEvent;
use app\events\UserCreatedHandler;
use app\queries\GetUserHandler;
use app\queries\GetUserQuery;
use mako\application\services\BusService as BaseBusService;

/**
 * Bus service.
 */
class BusService extends BaseBusService
{
    /**
     * {@inheritDocs}
     */
	protected function getCommandHandlers(): array
	{
		return
		[
			CreateUserCommand::class => CreateUserHandler::class,
		];
	}

    /**
     * {@inheritDocs}
     */
    protected function getEventHandlers(): array
	{
		return
		[
			UserCreatedEvent::class => UserCreatedHandler::class,
		];
	}

    /**
     * {@inheritDocs}
     */
    protected function getQueryHandlers(): array
	{
		return
		[
			GetUserQuery::class => GetUserHandler::class,
		];
	}
}

```