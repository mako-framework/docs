# Routing

--------------------------------------------------------

* [Basics](#basics)
* [Route parameters](#route_parameters)
* [Route middleware](#route_middleware)
	- [Defining middleware](#route_middleware:defining_middleware)
	- [Assigning middleware](#route_middleware:assigning_middleware)
* [Route groups](#route_groups)
* [Reverse routing](#reverse_routing)
* [Faking request methods](#faking_request_methods)

--------------------------------------------------------

The Mako router lets you map URL patterns to class methods and closures. It also allows you to perform reverse routing so that you don't have to hardcode URLs in your application.

Routes are registered in the ```app/routing/routes.php``` file and there are three variables avaiable in the scope, ```$routes``` (the route collection) and ```$app``` (the application instance) and ```$container``` (the IoC container instance).

--------------------------------------------------------

<a id="basics"></a>

### Basics

The following route will forward all ```GET``` requests to the ```/``` route to the ```welcome``` method of the ```app\controllers\Home```controller class.

	$routes->get('/', 'app\controllers\Home::welcome');

If you want the route to respond to ```POST``` requests instead then you'll have to use the ```post``` method.

	$routes->post('/', 'app\controllers\Home::welcome');

The available methods are ```get```, ```post```, ```put```, ```patch```, and ```delete```.

You can also make a route respond to all request methods using the ```all``` method.

	$routes->all('/', 'app\controllers\Home::welcome');

> All routes will by default respond to requests made using the ```OPTIONS``` method. ```GET``` routes will also respond to ```HEAD``` requests.

If you only want to allow a specific set of methods then you can use the ```register``` method.

	$routes->register(['GET', 'POST'], '/', 'app\controllers\Home::welcome');

Routes can also execute closures instead of class methods.

	$routes->get('/hello-world', function()
	{
		return 'Hello, world!';
	});

--------------------------------------------------------

<a id="route_parameters"></a>

### Route parameters

You'll often want to send parameters to your route actions. This is easy and can be done like this.

	$routes->get('/articles/{id}', function($id)
	{
		return $id;
	});

If you need to make a parameter optional then you can do so by adding the ```?``` suffix.

	$routes->get('/articles/{id}/{slug}?', function($id, $slug = null)
	{
		return $id . ' ' . $slug;
	});

You can also impose constraints on your parameters using the ```when``` method. The route will not be matched unless all constraints are satisfied.

	$routes->get('/articles/{id}', function($id)
	{
		return 'article ' . $id;
	})
	->when(['id' => '[0-9]+']);

Closure actions get executed by the ```Container::call()``` method so all dependecies are automatically [injected](:base_url:/docs/:version:/getting-started:dependency-injection).

	$routes->get('/article/{id}', function(ViewFactory $view, $id)
	{
		return $view->render('article', ['id' => $id]);
	})
	->when(['id' => '[0-9]+']);

--------------------------------------------------------

<a id="route_middleware"></a>

### Route middleware

<a id="route_middleware:defining_middleware"></a>

Route middleware allows you to alter the request and reponse both before and after a route action gets executed.

![middleware](:base_url:/assets/img/middleware.png)

#### Defining middleware

Middleware should (but is not required to) implement the ```MiddlewareInterface```. The example below is the most basic middleware implementation (it doesn't actually do anything).

	<?php

	namespace app\routing\middleware;

	use Closure;

	use mako\http\Request;
	use mako\http\Response;
	use mako\http\routing\middleware\MiddlewareInterface;

	class PassthroughMiddleware implements MiddlewareInterface
	{
		public function execute(Request $request, Response $response, Closure $next): Response
		{
			return $next($request, $response);
		}
	}

Middleware has to be registered in the ```app/routing/middleware.php``` file before you can use them. There are three variables avaiable in the scope, ```$middleware``` (the middleware collection), ```$app``` (the application instance) and ```$container``` (the IoC container instance).

	$middleware->register('passthrough', PassthroughMiddleware::class);

In the next example we'll create a middleware that returns a cached response if possible.

Note that all middleware is instantiated through the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor.

	<?php

	namespace app\routing\middleware;

	use Closure;

	use mako\cache\CacheManager;
	use mako\http\Request;
	use mako\http\Response;
	use mako\http\routing\middleware\MiddlewareInterface;

	class CacheMiddleware implements MiddlewareInterface
	{
		protected $cache;

		protected $minutes;

		public function __construct(CacheManager $cache, $minutes = null)
		{
			$this->cache = $cache;

			$this->minutes = $minutes ?? 10;
		}

		public function execute(Request $request, Response $response, Closure $next): Response
		{
			if(this->cache->has('route.' . $request->path()))
			{
				return $response->body($cache->get('route.' . $request->path()));
			}

			$reponse = $next($request, $response);

			$this->cache->put('route.' . $request->path(), $response->getBody(), 60 * $this->minutes);

			return $response;
		}
	}

> The cache example above is very basic and should probably not be used in a production environment.

<a id="route_middleware:assigning_middleware"></a>

#### Assigning middleware

Assigning middleware to a route is done using the ```middleware``` method. You can also pass an array of middleware if your route requires multiple middlewares. Middleware will get executed in the order that they are assigned.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->when(['id' => '[0-9]+'])
	->middleware('cache');

You can also pass parameters to your middleware. In the example below we're telling the middleware to cache the response for 60 minutes instead of the default 10.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->when(['id' => '[0-9]+'])
	->middleware('cache:{"minutes":60}');

> Anything after the first colon symbol (```:```) will be parsed as JSON. In the example above we're telling the dispatcher that the value ```60``` should be passed to the ```$minutes``` parameter of the middleware constructor.

--------------------------------------------------------

<a id="route_groups"></a>

### Route groups

Route groups are usefull when you have a set of routes with the same constraints and middleware.

	$options =
	[

		'middlware' => 'cache',
		'when'      => ['id' => '[0-9]+'],
		'namespace' => 'app\controllers',
	];

	$routes->group($options, function($routes)
	{
		$routes->get('/articles/{id}', 'Articles::view');

		$routes->get('/photos/{id}', 'Photos::view');
	});

All routes within the group will now have the same middleware and constraints. You can also nest groups if needed.

The following options are available when creating a route group. They are also available as chainable methods on individual routes.

| Option      | Method       | Description                                              |
|-------------|--------------|----------------------------------------------------------|
| middleware  | middleware   | A middleware or an array of middlewares                  |
| namespace   | namespace    | The controller namespace (closures will not be affected) |
| prefix      | prefix       | Route prefix                                             |
| when        | when         | An array of parameter constraints                        |

--------------------------------------------------------

<a id="reverse_routing"></a>

### Reverse routing

You can assign names to your routes when you define them. This will allow you to perform reverse routing, thus removing the need of hardcoding URLs in your views.

	$routes->get('/', 'Home::Welcome', 'home');

The route in the example above has been named ```home``` and we can now create a URL to the route using the ```toRoute``` method of the ```URLBuilder``` class.

	<a href="{{$urlBuilder->toRoute('home')}}">Home</a>

You can also pass parameters to your route URLs.

	<a href="{{$urlBuilder->toRoute('articles.view', ['id' => 1])">Article</a>

--------------------------------------------------------

<a id="faking_request_methods"></a>

### Faking request methods

Most browsers only support sending ```GET``` and ```POST``` requests. You can get around this limitation by performing a ```POST``` request including a ```REQUEST_METHOD_OVERRIDE``` field where you specify the request method you want to use.

	<input type="hidden" name="REQUEST_METHOD_OVERRIDE" value="DELETE">

Another solution is to send a ```X_HTTP_METHOD_OVERRIDE``` header.