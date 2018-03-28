# Events

--------------------------------------------------------

* [Event listener](#event_listener)

--------------------------------------------------------

Mako includes two ways of handling events, an event listener and a trait that makes your class observable.

--------------------------------------------------------

<a id="event_listener"></a>

### Event listener

The `register` method lets you register an event handler that will get executed when the event is triggered. You can register multiple handlers for the same event. They will be executed in the order that they were registered.

```
$this->event->register('foobar', function()
{
	return 'foobar event handler';
});
```

You can also handle your events using a class instead of a closure.

```
$this->event->register('foobar', FoobarHandler::class);
```

Class handlers will be instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor. Both closure and class handlers will be executed by the `Container::call()` and all dependencies will automatically be [injected](:base_url:/docs/:version:/getting-started:dependency-injection) there as well.

```
<?php

namespace app\events;

use mako\event\EventHandlerInterface;

class FoobarHandler implements EventHandlerInterface
{
	public function handle()
	{
		return 'foobar event handler';
	}
}
```

The `has` method will return TRUE if an event handler has been registered and FALSE if not.

```
$registered = $this->event->has('foobar');
```

The `clear` method will clear all event handlers registered for an event.

```
$this->event->clear('foobar');
```

The `override` method will clear all previously registered handlers for an event before registering a new handler.

```
$this->event->override('foobar', function()
{
	return 'foobar event 1';
});
```

The `trigger` method executes all handlers for the registered event and returns an array containing all the return values.

```
$values = $this->event->trigger('foobar');
```

You can also pass arguments your handlers using the optional second parameter.

```
$values = $this->event->trigger('foobar', [1, 2, 3]);
```

The third optional parameter lets you stop event handling if one of the handlers return `FALSE`.

```
$values = $this->event->trigger('foobar', [1, 2, 3], true);
```
