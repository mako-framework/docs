# Routing

--------------------------------------------------------

* [Basics](#basics)
* [Route parameters](#route_parameters)
* [Route middleware](#route_middleware)
	- [Defining middleware](#route_middleware:defining_middleware)
	- [Assigning middleware](#route_middleware:assigning_middleware)
	- [Middleware priority](#route_middleware:middleware_priority)
* [Route constraints](#route_constraints)
	- [Defining constraints](#route_constraints:defining_constraints)
	- [Assigning constraints](#route_constraints:assigning_constraints)
* [Route groups](#route_groups)
* [Reverse routing](#reverse_routing)
* [Faking request methods](#faking_request_methods)

--------------------------------------------------------

The Mako router lets you map URL patterns to class methods and closures. It also allows you to perform reverse routing so that you don't have to hardcode URLs in your application.

Routes are registered in the `app/routing/routes.php` file and there are three variables available in the scope, `$routes` (the route collection) and `$app` (the application instance) and `$container` (the container instance).

--------------------------------------------------------

<a id="basics"></a>

### Basics

The following route will forward all `GET` requests to the `/` route to the `welcome` method of the `app\controllers\Home`controller class.

```
$routes->get('/', [Home::class, 'welcome']);
```

If you want the route to respond to `POST` requests instead then you'll have to use the `post` method.

```
$routes->post('/', [Home::class, 'welcome']);
```

The available methods are `get`, `post`, `put`, `patch`, and `delete`.

You can also make a route respond to all request methods using the `all` method.

```
$routes->all('/', [Home::class, 'welcome']);
```

> All routes respond to requests made using the `OPTIONS` request method. `GET` routes will also respond to `HEAD` requests.

It is also possible to register a route that responds to a custom set of request methods using the `register` method.

```
$routes->register(['GET', 'POST'], '/', [Home::class, 'welcome']);
```

As previously mentioned, routes can also point to closures instead of class methods.

```
$routes->get('/hello-world', function()
{
	return 'Hello, world!';
});
```

--------------------------------------------------------

<a id="route_parameters"></a>

### Route parameters

You'll often want to send parameters to your route actions. This can easily be achieved using the following syntax.

```
$routes->get('/articles/{id}', function($id)
{
	return $id;
});
```

Parameters can be marked as optional by suffixing them with a question mark (`?`).

```
$routes->get('/articles/{id}/{slug}?', function($id, $slug = null)
{
	return $id . ' ' . $slug;
});
```

Parameters will match any character except for slashes (`/`); however, you can define your own custom parameter patterns using the `patterns` method.

```
$routes->get('/articles/{id}', function($id)
{
	return 'article ' . $id;
})
->patterns(['id' => '[0-9]+']);
```

Closure actions get executed by the `Container::call()` method so all dependencies are automatically [injected](:base_url:/docs/:version:/getting-started:dependency-injection).

```
$routes->get('/article/{id}', function(ViewFactory $view, $id)
{
	return $view->render('article', ['id' => $id]);
})
->patterns(['id' => '[0-9]+']);
```

--------------------------------------------------------

<a id="route_middleware"></a>

### Route middleware

Route middleware allows you to alter the request and response both before and after a route action gets executed.

![middleware](:base_url:/assets/img/middleware.png)

<a id="route_middleware:defining_middleware"></a>

#### Defining middleware

The example below is the most basic middleware implementation (it doesn't actually do anything).

```
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
```

Middleware has to be registered in the `app/routing/middleware.php` file before you can use them. There are three variables available in the scope, `$dispatcher` (the route dispatcher), `$app` (the application instance) and `$container` (the container instance).

```
$dispatcher->registerMiddleware('passthrough', PassthroughMiddleware::class);
```

In the next example we'll create a middleware that returns a cached response if possible.

Note that all middleware is instantiated through the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor.

```
<?php

namespace app\routing\middleware;

use Closure;
use mako\cache\CacheManager;
use mako\http\Request;
use mako\http\Response;
use mako\http\routing\middleware\MiddlewareInterface;

class CacheMiddleware implements MiddlewareInterface
{
	protected $minutes;

	protected $cache;

	public function __construct(int $minutes = 10, CacheManager $cache)
	{
		$this->minutes = $minutes;

		$this->cache = $cache;
	}

	public function execute(Request $request, Response $response, Closure $next): Response
	{
		if(this->cache->has('route.' . $request->getPath()))
		{
			return $response->setBody($cache->get('route.' . $request->getPath()));
		}

		$response = $next($request, $response);

		$this->cache->put('route.' . $request->getPath(), $response->getBody(), 60 * $this->minutes);

		return $response;
	}
}
```

> The cache example above is very basic and should probably not be used in a production environment.
{.warning}

<a id="route_middleware:assigning_middleware"></a>

#### Assigning middleware

Assigning middleware to a route is done using the `middleware` method. You can also pass an array of middleware if your route requires multiple middleware. Middleware will get executed in the order that they are assigned.

```
$routes->get('/articles/{id}', [Articles::class, 'view'])
->patterns(['id' => '[0-9]+'])
->middleware('cache');
```

You can also pass parameters to your middleware. Parameters are parsed as JSON so booleans, strings, arrays, objects (associative arrays), null and numeric values are valid.

In the example below we're telling the middleware to cache the response for 60 minutes instead of the default 10.

```
$routes->get('/articles/{id}', [Articles::class, 'view'])
->patterns(['id' => '[0-9]+'])
->middleware('cache("minutes":60)');
```

> In the example above we created a middleware that uses named parameters; however, you can also use unnamed parameters (`cache(60)`).
> Note that if you use unnamed parameters then you'll have to make sure that the order of the parameters is correct.

If you have middleware that you want to assign to all your routes then you can set them as global.

```
$dispatcher->setMiddlewareAsGlobal(['cache']);
```

<a id="route_middleware:middleware_priority"></a>

#### Middleware priority

As mentioned above, middleware get executed in the order that they are assigned to the route. You can dictate the execution order by configuring middleware priority.

You can set the middleware priority while registering the middleware using the optional third parameter of the `registerMiddleware` method.

```
$dispatcher->registerMiddleware('cache', CacheMiddleware::class, 1);
$dispatcher->registerMiddleware('passthrough', PassthroughMiddleware::class, 2);
```

Or you can set the priority of all your middleware using the `setMiddlewarePriority` method.

```
$dispatcher->setMiddlewarePriority(['cache' => 1, 'passthrough' => 2]);
```

In both examples above we're making sure that the `cache` middleware gets executed first, followed by the `passthrough` middleware.

You can use middleware priority without having to configure all your middleware. Non-configured middleware will be assigned a default priority of `100`. This means that if you have a middleware that you want executed last then you can set its priority to a value of `101` or above.

--------------------------------------------------------

<a id="route_constraints"></a>

### Route constraints

Route constraints allow you to set additional requirements that must be met before a route is matched.

<a id="route_constraints:defining_constraints"></a>

#### Defining constraints

Note that all constraints are instantiated through the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor.

The following constraint will match a route if the `X-Api-Version` header matches the desired version. A default value of `2.0` is assumed if no header is present.

```
<?php

namespace app\routing\constraints;

use mako\http\Request;
use mako\http\routing\constraints\ConstraintInterface;

class ApiVersionConstraint implements ConstraintInterface
{
	protected $version;

	protected $request;

	public function __construct(string $version, Request $request)
	{
		$this->version = $version;

		$this->request = $request;
	}

	public function isSatisfied(): bool
	{
		return $this->request->getHeaders()->get('X-Api-Version', '2.0') === $this->version;
	}
}
```

Constraints have to be registered in the `app/routing/constraints.php` file before you can use them. There are three variables available in the scope, `$router` (the router), `$app` (the application instance) and `$container` (the container instance).

```
$router->registerConstraint('api_version', ApiVersionConstraint::class);
```

<a id="route_constraints:assigning_constraints"></a>

#### Assigning constraints

Assigning constraints to a route is done using the `constraint` method. You can also pass an array of constraints if your route requires multiple constraints.

```
$routes->get('/', [Api2::class, 'index'])->constraint('api_version("2.0")');
$routes->get('/', [Api1::class, 'index'])->constraint('api_version("1.0")');
```

The first route will be matched if no `X-Api-Version` header is present or if the value equals `2.0`. The second route will be matched if the header value is set to `1.0`.

> In the example above we used unnamed parameters for our constraints. You can also use named constraints (`api_version("version":"2.0")`).

If you have constraints that you want to assign to all your routes then you can set them as global.

```
$router->setConstraintAsGlobal(['api_version("2.0")']);
```

--------------------------------------------------------

<a id="route_groups"></a>

### Route groups

Route groups are useful when you have a set of routes with the same settings.

```
$options =
[

	'middleware' => 'cache',
	'patterns'   => ['id' => '[0-9]+'],
];

$routes->group($options, function($routes)
{
	$routes->get('/articles/{id}', [Articles::class, 'view']);

	$routes->get('/photos/{id}', [Photos::class, 'view']);
});
```

All routes within the group will now use the same middleware and regex pattern. You can also nest groups if needed.

The following options are available when creating a route group. They are also available as chainable methods on individual routes.

| Option      | Method       | Description                                              |
|-------------|--------------|----------------------------------------------------------|
| middleware  | middleware   | A middleware or an array of middleware                   |
| constraint  | constraint   | A constraint or an array of constraints                  |
| prefix      | prefix       | Route prefix                                             |
| patterns    | patterns     | An array of parameter regex patterns                     |

--------------------------------------------------------

<a id="reverse_routing"></a>

### Reverse routing

You can assign names to your routes when you define them. This will allow you to perform reverse routing, thus removing the need of hardcoding URLs in your project.

```
$routes->get('/', [Home::class, 'welcome'], 'home');
```

The route in the example above has been named `home` and we can now create a URL to the route using the `toRoute` method of the `URLBuilder` class.

```
<a href="{{$urlBuilder->toRoute('home')}}">Home</a>
```

You can also pass parameters to your route URLs.

```
<a href="{{$urlBuilder->toRoute('articles.view', ['id' => 1])">Article</a>
```

--------------------------------------------------------

<a id="faking_request_methods"></a>

### Faking request methods

Most browsers only support sending `GET` and `POST` requests. You can get around this limitation by performing a `POST` request including a `REQUEST_METHOD_OVERRIDE` field where you specify the request method you want to use.

```
<input type="hidden" name="REQUEST_METHOD_OVERRIDE" value="DELETE">
```

Another solution is to send a `X_HTTP_METHOD_OVERRIDE` header.
