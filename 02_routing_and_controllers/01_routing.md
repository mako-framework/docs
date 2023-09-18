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

### <a id="basics" href="#basics">Basics</a>

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
$routes->get('/hello-world', fn () => 'Hello, world!');
```

--------------------------------------------------------

### <a id="route_parameters" href="#route_parameters">Route parameters</a>

You'll often want to send parameters to your route actions. This can easily be achieved using the following syntax.

```
$routes->get('/articles/{id}', fn ($id) => $id);
```

Parameters can be marked as optional by suffixing them with a question mark (`?`).

```
$routes->get('/articles/{id}/{slug}?', fn ($id, $slug = null) => $id . ' ' . $slug);
```

Parameters will match any character except for slashes (`/`); however, you can define your own custom parameter patterns using the `patterns` method.

```
$routes->get('/articles/{id}', fn ($id) => 'article ' . $id)
->patterns(['id' => '[0-9]+']);
```

Both class method and closure actions get executed by the `Container::call()` method so all dependencies are automatically [injected](:base_url:/docs/:version:/getting-started:dependency-injection).

```
$routes->get('/article/{id}', fn (ViewFactory $view, $id) => $view->render('article', ['id' => $id]))
->patterns(['id' => '[0-9]+']);
```

--------------------------------------------------------

### <a id="route_middleware" href="#route_middleware">Route middleware</a>

Route middleware allows you to alter the request and response both before and after a route action gets executed.

<img src=":base_url:/assets/img/middleware.svg" style="width:50%;min-width:340px;display:block;margin-left:auto;margin-right:auto;" />

#### <a id="route_middleware:defining_middleware" href="#route_middleware:defining_middleware">Defining middleware</a>

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
	public function __construct(
		protected CacheManager $cache, 
		protected int $minutes = 10
	)
	{}

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

#### <a id="route_middleware:assigning_middleware" href="#route_middleware:assigning_middleware">Assigning middleware</a>

Assigning middleware to a route can be done using the `middleware` method. If your route requires multiple middleware then you can chain multiple calls to the `middleware` method.

```
$routes->get('/articles/{id}', [Articles::class, 'view'])
->patterns(['id' => '[0-9]+'])
->middleware(CacheMiddleware::class);
```

You can also pass parameters to your middleware. In the example below we're telling the middleware to cache the response for 60 minutes instead of the default 10.

```
$routes->get('/articles/{id}', [Articles::class, 'view'])
->patterns(['id' => '[0-9]+'])
->middleware(CacheMiddleware::class, minutes: 60);
```

Middleware can also be assigned using the `Middleware` attribute on route action classes and methods.

```
#[Middleware(CacheMiddleware::class, minutes: 60)]
public function myAction(): string
{
	return 'Hello, world!';
}
```

If you have middleware that you want to assign to all your routes then you can register them as global in the `app/routing/middleware.php` file. There are three variables available in the scope, `$dispatcher` (the route dispatcher), `$app` (the application instance) and `$container` (the container instance).

```
$dispatcher->registerGlobalMiddleware(CacheMiddleware::class, minutes: 60);
```

#### <a id="route_middleware:middleware_priority" href="#route_middleware:middleware_priority">Middleware priority</a>

As mentioned above, middleware get executed in the order that they are assigned to the route. You can dictate the execution order by configuring middleware priority using the `setMiddlewarePriority` method.

```
$dispatcher->setMiddlewarePriority(CacheMiddleware::class, 1);
$dispatcher->setMiddlewarePriority(PassthroughMiddleware::class, 2);
```

In both examples above we're making sure that the `cache` middleware gets executed first, followed by the `passthrough` middleware.

You can use middleware priority without having to configure all your middleware. Non-configured middleware will be assigned a default priority of `100`. This means that if you have a middleware that you want executed last then you can set its priority to a value of `101` or above.

--------------------------------------------------------

### <a id="route_constraints" href="#route_constraints">Route constraints</a>

Route constraints allow you to set additional requirements that must be met before a route is matched.

#### <a id="route_constraints:defining_constraints" href="#route_constraints:defining_constraints">Defining constraints</a>

Note that all constraints are instantiated through the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor.

The following constraint will match a route if the `X-Api-Version` header matches the desired version. A default value of `2.0` is assumed if no header is present.

```
<?php

namespace app\routing\constraints;

use mako\http\Request;
use mako\http\routing\constraints\ConstraintInterface;

class ApiVersionConstraint implements ConstraintInterface
{
	public function __construct(
		protected string $version, 
		protected Request $request
	)
	{}

	public function isSatisfied(): bool
	{
		return $this->request->getHeaders()->get('X-Api-Version', '2.0') === $this->version;
	}
}
```

#### <a id="route_constraints:assigning_constraints" href="#route_constraints:assigning_constraints">Assigning constraints</a>

Assigning constraints to a route is done using the `constraint` method. If your route requires multiple constraints then you can chain multiple calls to the `constraint` method.

```
$routes->get('/', [Api2::class, 'index'])->constraint(ApiVersionConstraint::class, version: '2.0');
$routes->get('/', [Api1::class, 'index'])->constraint(ApiVersionConstraint::class, version: '1.0');
```

The first route will be matched if no `X-Api-Version` header is present or if the value equals `2.0`. The second route will be matched if the header value is set to `1.0`.

> In the example above we used unnamed parameters for our constraints. You can also use named constraints (`api_version("version":"2.0")`).

Constraints can also be assigned using the `Constraint` attribute on route action classes and methods.

```
#[Constraint(ApiVersionConstraint::class, version: '2.0')]
public function myAction(): string
{
	return 'Hello, world!';
}
```

If you have constraints that you want to assign to all your routes then you can register them as global in the `app/routing/constraints.php` file. There are three variables available in the scope, `$router` (the router), `$app` (the application instance) and `$container` (the container instance).

```
$router->registerGlobalConstraint(ApiVersionConstraint::class, version: '2.0');
```

--------------------------------------------------------

### <a id="route_groups" href="#route_groups">Route groups</a>

Route groups are useful when you have a set of routes with the same settings.

```
$options =
[
	'middleware' => [CacheMiddleware::class],
	'patterns'   => ['id' => '[0-9]+'],
];

$routes->group($options, function ($routes)
{
	$routes->get('/articles/{id}', [Articles::class, 'view']);

	$routes->get('/photos/{id}', [Photos::class, 'view']);
});
```

All routes within the group will now use the same middleware and regex pattern. You can also nest groups if needed.

You can also pass parameters to middleware and constraints in route groups by using the following nested array structure.

```
$options =
[
	'middleware' => [[CacheMiddleware::class, ['minutes' => 60]]],
	'patterns'   => ['id' => '[0-9]+'],
];
```

The following options are available when creating a route group. They are also available as chainable methods on individual routes.

| Option      | Method       | Description                                              |
|-------------|--------------|----------------------------------------------------------|
| middleware  | middleware   | A middleware or an array of middleware                   |
| constraint  | constraint   | A constraint or an array of constraints                  |
| prefix      | prefix       | Route prefix                                             |
| patterns    | patterns     | An array of parameter regex patterns                     |

--------------------------------------------------------

### <a id="reverse_routing" href="#reverse_routing">Reverse routing</a>

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

### <a id="faking_request_methods" href="#faking_request_methods">Faking request methods</a>

Most browsers only support sending `GET` and `POST` form requests. You can get around this limitation by performing a `POST` request including a `REQUEST_METHOD_OVERRIDE` field where you specify the request method you want to use.

```
<input type="hidden" name="REQUEST_METHOD_OVERRIDE" value="DELETE">
```

Another solution is to send a `X-Http-Method-Override` header.
