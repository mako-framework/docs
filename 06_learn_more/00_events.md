# Events

--------------------------------------------------------

* [Event listener](#event_listener)
* [Observable trait](#observable_trait)

--------------------------------------------------------

Mako includes two ways of handling events, an event listener and a trait that makes your class observable.

--------------------------------------------------------

<a id="event_listener"></a>

### Event listener

First we'll need to create a listener instance.

	$listener = new Listener();

The ```register``` method lets you register an event handler that will get executed when the event is triggered.

	$listener->register('foobar', function()
	{
		return 'foobar event 1';
	});

You can register multiple handlers for the same event. They will be executed in the order that they were registered.

	$listener->register('foobar', function()
	{
		return 'foobar event 2';
	});

The ```registered``` method will return TRUE if an event handler has been registered and FALSE if not.

	$registered = $listener->registered('foobar');

The ```clear``` method will clear all event handlers registered for an event.

	$listener->clear('foobar');

The ```override``` method will clear all previously registered handlers for an event before registering a new handler.

	$listener->override('foobar', function()
	{
		return 'foobar event 1';
	});

The ```trigger``` method run all handlers for the registered event and return an array containing all the return values.

	$values = $listener->trigger('foobar');

	// You can also pass an array of values

	$values = $listener->trigger('foobar', [1, 2, 3]);

--------------------------------------------------------

<a id="observable_trait"></a>

### Observable trait

Your class must use the ```ObservableTrait``` in order to be obserable.

	class Observable
	{
		use \mako\event\ObsevableTrait;
	}

	$observable = new Observable;

The ```attachObserver``` method allows you to attach an observer to your class. You can attach multiple observers for the same event.

	$observable->attachObserver('foobar', function()
	{
		return 'foobar event';
	});

	// You can also attach an observer instance

	$observable->attachObserver('foobar', new MyObserver);

	// Or specify the name of your observer class. Remember to include the full namespace!

	$observable->attachObserver('foobar', '\app\observers\MyObserver');

> Observer classes must implement a public method named ```update``` that'll handle the event.

The ```hasObserver``` method returns TRUE if an event has an observer and FALSE if not.

	$observed = $observable->hasObserver('foobar');

The ```detachObserver``` method allows you to detach an observer from an event.

	$observable->detachObserver('foobar', new MyObserver);

	$observable->detachObserver('foobar', '\app\observers\MyObserver');

> The method does not work with closure observers.

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