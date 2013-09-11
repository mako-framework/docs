# Events

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The event class lets you register event listeners and trigger them.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```register``` method lets you register an event listener that will run when triggered.

	Event::register('foobar', function()
	{
		return 'foobar event 1';
	});

	// You can register multiple handlers for the same event.
	// They will be run in the order they were registered.

	Event::register('foobar', function()
	{
		return 'foobar event 2';
	});

The ```registered``` method will return TRUE if an event listener has been registered and FALSE if not.

	$registered = Event::registered('foobar'); // True

The ```clear``` method will clear all event listeners registered for an event.

	Event::clear('foobar');

The ```override``` method will clear all previously registered handlers for an event before registering a new handler.

	Event::override('foobar', function()
	{
		return 'foobar event 1';
	});

The ```trigger``` method run all handlers for the registered event and return an array containing all the return values.

	$values = Event::trigger('foobar');

	// You can also pass an array of values

	$values = Event::trigger('foobar', array(1, 2, 3));

The ```first``` method run all handlers for the registered event and return the result of the first one.

	$value = Event::first('foobar');

	// You can also pass an array of values

	$values = Event::first('foobar', array(1, 2, 3));