# Routing

--------------------------------------------------------

* [Basics](#basics)
* [Route parameters](#route_parameters)
* [Route filters](#route_filters)
* [Route groups](#route_groups)
* [Reverse routing](#reverse_routing)
* [Faking request methods](#faking_request_methods)

--------------------------------------------------------

The Mako router allows you to map URLs to controller and closure actions. It also allows you to perform reverse routing so that you don't have to hardcode URLs in your views.

Routes are registered in the ```app/routes.php``` file. There are three variables avaiable in the scope, ```$routes``` (the route collection) and ```$app``` (the application instance) and ```$container``` (the IoC container instnace).

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

If you only want to allow a specific set of methods then you can use the ```methods``` method.

	$routes->methods(['GET', 'POST'], '/', 'app\controllers\Home::welcome');

Routes can also execute closures instead of class methods. The two first parameters passed to the closures are the ```request``` and ```response``` objects. Any subsequent parameters will be route parameters.

	$routes->get('/hello-world', function($request, $response)
	{
		return 'Hello, world!';
	});

You can access the container instance in your closures like this.

	$routes->get('/hello-world', function($request, $response) use ($container)
	{
		return 'Hello, world!';
	});

--------------------------------------------------------

<a id="route_parameters"></a>

### Route parameters

You'll often want to send parameters to your route actions. This is easy and can be done like this.

	$routes->get('/articles/{id}', function($request, $response, $id)
	{
		return $id;
	});

If you need to make a parameter optional then you can do so by adding the ```?``` suffix.

	$routes->get('/articles/{id}/{slug}?', function($request, $response, $id, $slug = null)
	{
		return $id . ' ' . $slug;
	});

You can also impose constraints on your parameters using the ```constraints``` method. The route will not be matched unless all constraints are satisfied.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->constraints(['id' => '[0-9]+']);

--------------------------------------------------------

<a id="route_filters"></a>

### Route filters

You can define filters that will get executed before and after your route actions.

> The route action and ```after``` filters will **not** be executed if a ```before``` filter returns data.

	// Return cached version of route response if it's available

	$routes->filter('cache.get', function($request, $response) use ($container)
	{
		if($container->get('cache')->has('route.' . $request->path()))
		{
			return $container->get('cache')->get('route.' . $request->path());
		}
	});

	// Cache route response for 10 minutes

	$routes->filter('cache.put', function($request, $response, $minutes = 10) use ($container)
	{
		$container->get('cache')->put('route.' . $request->path(), $response->getBody(), 60 * $minutes);
	});

> The cache example above is very basic and should probably not be used in a production environment.

Assigning filters to a route is done using the ```before``` and ```after``` methods. You can also pass an array of filters if your route has multiple filters. The filters get executed in the order that they are assigned.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->constraints(['id' => '[0-9]+'])
	->before('cache.read')
	->after('cache.write');

You can also pass parameters to your filters. Multiple parameters are separated by a comma. In the example below we're telling the filter to cache the response for 60 minutes instead of the default 10.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->constraints(['id' => '[0-9]+'])
	->before('cache.read')
	->after('cache.write[60]');

--------------------------------------------------------

<a id="route_groups"></a>

### Route groups

Route groups are usefull when you have a set of groups with the same constraints and filters.

	$options = ['before' => 'cache.read', 'after' => 'cache.write', 'constraints' => ['id' => '[0-9]+']];

	$routes->group($options, function($routes)
	{
		$routes->get('/articles/{id}', 'app\controllers\Articles::view');

		$routes->get('/photos/{id}', 'app\controllers\Photos::view');
	});

All routes within the group will now have the same filters and constraints. You can also nest groups if needed. 

The following options are available when creating a route group. They are also available as chainable methods on individual routes.

| Option      | Description                                                                              |
|-------------|------------------------------------------------------------------------------------------|
| before      | A before filter or an array of before filters                                            |
| after       | An after filter or an array of aflter filters                                            |
| constraints | An array of parameter constraints                                                        |
| prefix      | Route prefix                                                                             |
| headers     | An array of response headers (the key is the header field and value is the header value) |

--------------------------------------------------------

<a id="reverse_routing"></a>

### Reverse routing

You can assign names to your routes when you define them. This will allow you to perform reverse routing, thus removing the need of hardcoding URLs in your views.

	$routes->get('/', 'app\controllers\Home::Welcome', 'home');

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