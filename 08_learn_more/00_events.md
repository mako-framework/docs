# Events

--------------------------------------------------------

* [Event listener](#event_listener)
* [Observable trait](#observable_trait)

--------------------------------------------------------

Mako includes two ways of handling events, an event listener and a trait that makes your class observable.

--------------------------------------------------------

<a id="event_listener"></a>

### Event listener

The ```register``` method lets you register an event handler that will get executed when the event is triggered. You can register multiple handlers for the same event. They will be executed in the order that they were registered.

	$this->event->register('foobar', function()
	{
		return 'foobar event handler';
	});

You can also handle your events using a class instead of a closure.

	$this->event->register('foobar', 'app\events\FoobarHandler');

Class handlers will be instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor. Both closure and class handlers will be executed by the ```Container::call()``` and all dependencies will automatically be [injected](:base_url:/docs/:version:/getting-started:dependency-injection) there as well.

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

The ```has``` method will return TRUE if an event handler has been registered and FALSE if not.

	$registered = $this->event->has('foobar');

The ```clear``` method will clear all event handlers registered for an event.

	$this->event->clear('foobar');

The ```override``` method will clear all previously registered handlers for an event before registering a new handler.

	$this->event->override('foobar', function()
	{
		return 'foobar event 1';
	});

The ```trigger``` method run all handlers for the registered event and return an array containing all the return values.

	$values = $this->event->trigger('foobar');

You can also pass arguments your handlers using the optional second parameter.

	$values = $this->event->trigger('foobar', [1, 2, 3]);

The third optional parameter lets you stop event handling if one of the handlers return ```FALSE```.

	$values = $this->event->trigger('foobar', [1, 2, 3], true);

--------------------------------------------------------

<a id="observable_trait"></a>

### Observable trait

Your class must use the ```ObservableTrait``` in order to be obserable.

	class Observable
	{
		use mako\event\ObsevableTrait;
	}

	$observable = new Observable;

The ```attachObserver``` method allows you to attach an observer to your class. You can attach multiple observers for the same event.

	$observable->attachObserver('foobar', function()
	{
		return 'foobar event';
	});

The ```hasObserver``` method returns TRUE if an event has an observer and FALSE if not.

	$observed = $observable->hasObserver('foobar');

The ```clearObservers``` will clear all observers. You can also specify which event you want to clear.

	$observable->clearObservers();

	$observable->clearObservers('foobar');

The ```overrideObservers``` method clears all observers for an event and registers a new one.

	$observable->overrideObservers('foobar', function($string)
	{
		return mb_strtoupper($string);
	});

The ```notifyObservers``` method will notify all observers of an event and return an array containing all of their return values.

	$returnValues = $this->notifyObservers('foobar', 'hello, world!');

> The ```notifyObservers``` method is ```protected``` and can only be called from within the observable class.