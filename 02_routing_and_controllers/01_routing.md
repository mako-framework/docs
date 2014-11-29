# Routing

--------------------------------------------------------

* [Basics](#basics)
* [Route parameters](#route_parameters)
* [Route filters](#route_filters)
	- [Defining filters](#route_filters:defining_filters)
	- [Assigning filters](#route_filters:assigning_filters)
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

If you only want to allow a specific set of methods then you can use the ```methods``` method.

	$routes->methods(['GET', 'POST'], '/', 'app\controllers\Home::welcome');

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

<a id="route_filters"></a>

### Route filters

<a id="route_filters:defining_filters"></a>

#### Defining filters

You can define filters that will get executed before and after your route actions.

Filters are registered in the ```app/routing/filters.php``` file. There are three variables avaiable in the scope, ```$filters``` (the filter collection) and ```$app``` (the application instance) and ```$container``` (the IoC container instance).

Closure filters get executed by the ```Container::call()``` method so all dependecies are automatically [injected](:base_url:/docs/:version:/getting-started:dependency-injection).

The route filters (both class filters and closures) are executed using the ```Container::call()``` method. This means that you typehint dependencies just like you can with route actions.

> Note that a route action and its after filters will **not** be executed if a before filter returns data.

This filter will return cached version of route response if it's available.

	$filters->register('cache.get', function(Request $request, CacheManager $cache)
	{
		if($cache->has('route.' . $request->path()))
		{
			return $cache->get('route.' . $request->path());
		}
	});

This filter will cache route responses for 10 minutes:

	$filters->register('cache.put', function(Request $request, Response $response, CacheManager $cache, $minutes = 10)
	{
		$cache->put('route.' . $request->path(), $response->getBody(), 60 * $minutes);
	});

> The cache example above is very basic and should probably not be used in a production environment.

You can also create a filter classes instead of closures. The class will be instantiated through the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject your dependencies through the constructor.

	namespace app\routing\filters;

	use \mako\http\Request;
	use \mako\http\Response;

	class MyFilter
	{
		public function construct(Request $request, Response $response)
		{
			$this->request = $request;

			$this->response = $response;
		}

		public function filter()
		{
			// Do your filtering here
		}
	}

Registering a filter class is just as easy as registering a closure.

	$filters->register('my_filter', 'app\routing\filters\MyFilter');

<a id="route_filters:assigning_filters"></a>

#### Assigning filters

Assigning filters to a route is done using the ```before``` and ```after``` methods. You can also pass an array of filters if your route has multiple filters. The filters get executed in the order that they are assigned.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->when(['id' => '[0-9]+'])
	->before('cache.read')
	->after('cache.write');

You can also pass parameters to your filters. In the example below we're telling the filter to cache the response for 60 minutes instead of the default 10.

	$routes->get('/articles/{id}', 'app\controllers\Articles::view')
	->when(['id' => '[0-9]+'])
	->before('cache.read')
	->after('cache.write:{"minutes":60}');

> Anything after the first colon symbol (```:```) will be parsed as JSON. In the example above we're telling the dispatcher that the value ```60``` should be passed to the ```$minutes``` parameter of the filter.

--------------------------------------------------------

<a id="route_groups"></a>

### Route groups

Route groups are usefull when you have a set of groups with the same constraints and filters.

	$options = 
	[

		'before'    => 'cache.read', 
		'after'     => 'cache.write', 
		'when'      => ['id' => '[0-9]+'],
		'namespace' => 'app\controllers',
	];

	$routes->group($options, function($routes)
	{
		$routes->get('/articles/{id}', 'Articles::view');

		$routes->get('/photos/{id}', 'Photos::view');
	});

All routes within the group will now have the same filters and constraints. You can also nest groups if needed. 

The following options are available when creating a route group. They are also available as chainable methods on individual routes.

| Option      | Method       | Description                                                                              |
|-------------|--------------|------------------------------------------------------------------------------------------|
| before      | before       | A before filter or an array of before filters                                            |
| after       | after        | An after filter or an array of aflter filters                                            |
| when        | when         | An array of parameter constraints                                                        |
| prefix      | prefix       | Route prefix                                                                             |
| namespace   | setNamespace | The controller namespace (closures will not be affected)                                 |

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