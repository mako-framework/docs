# Upgrading

--------------------------------------------------------

* [Application](#application)

* [Framework](#framework)
	- [Routing](#framework:routing)
	- [HTTP Middleware](#framework:http_middleware)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```5.2.x``` to ```5.3.x```.

--------------------------------------------------------

<a id="application"></a>

### Application

You'll have to create a `constraints.php` file and a `constraints` directory in your `app/routing` directory (remember to add a `.gitkeep` file in the `constraints` directory).

--------------------------------------------------------

<a id="framework"></a>

### Framework

<a id="framework:routing"></a>

#### Routing

The `when` method has been renamed to `patterns`.

<a id="framework:http_middleware"></a>

#### HTTP Middleware

All HTTP middleware must now implement the `mako\http\routing\middleware\MiddlewareInterface`. An abstract class (`mako\http\routing\middleware\Middleware`) implementing parts of the interface is included.

Middleware parameters are no longer injected through the constructor but instead via a the `setParameters` method (implemented by the abstract base class). Classes extending the included abstract base middleware can also retrieve parameters using the `getParameter` method.

Middleware is now registered with the route dispatcher.

	// Old

	$middleware->register('foo', Foo::class);

	// New

	$dispatcher->registerMiddleware('foo', Foo::class);

Middleware priority is now registered with the route dispatcher.

	// Old

	$middleware->setPriority(['foo' => 1]);

	// New

	$dispatcher->setMiddlewarePriority(['foo' => 1]);

See the routing docs for more details.
