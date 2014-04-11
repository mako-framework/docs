# Dependency injection

--------------------------------------------------------

* [Basics](#basics)
* [Services](#services)
* [Proxies](#proxies)

--------------------------------------------------------

Mako 4 comes with a inversion of controler container that allows you to inject dependencies into your controllers and command line tasks. Using dependency injection makes your application more maintainable and decoupled. Another benefit of using the dependency injection pattern is that it greatly increases the testability of the code, thus making it less prone to bugs.

--------------------------------------------------------

<a id="basics"></a>

### Basics

The IoC container is by default avaiable in the ```app/bootstrap.php``` and ```app/routes.php``` files.

#### Registering dependencies

The ```register``` method allows you to register a dependency in the container.

	$app->register('app\lib\FooInterface', 'app\lib\Foo');

You can also register a key along with the type hint to so that you can save a few keystrokes when resolving classes.

	$app->register(['app\lib\FooInterface', 'foo'], 'app\lib\Foo');

If your dependency requires constructor parameters then you can register it using a closure.

	$app->register(['app\lib\BarInterface', 'bar'], function($app)
	{
		return new \app\lib\Bar('parameter value');
	});

> The container will automatically try to resolve type hinted parameters.

The ```registerSingleton``` method does the same as the ```register``` method except that it makes sure that the same instance is returned every time the class is resolved through the container.

	$app->registerSingleton(['app\lib\BarInterface', 'bar'], function($app)
	{
		return new \app\lib\Bar('parameter value');
	});

The ```registerInstance``` method is similar to the ```registerSingleton``` method. The only difference is that it allows you to register an existing instance in the container.

	$app->registerInstance(['app\lib\BarInterface', 'bar'], new \app\lib\Bar('parameter value'));

The ```has``` method allows you to check for the presence of an item in the container.

	if($app->has('bar'))
	{
		// do something
	}

The ```get``` method lets you resolve a class through the IoC container.

	$foo = $app->get('foo');

The class does not have to be registered in the container to be resolvable.

	class Depends
	{
		protected $foo;
		protected $bar;

		public function __construct(\app\lib\FooInterface $foo, \app\lib\BarInterface $bar)
		{
			$this->foo = $foo;
			$this->bar = $bar;
		}
	}

We can now resolve the ```Depends``` class using the IoC container. Both its dependencies will automatically be injected.

	$depends = $app->get('Depends');

The ```getFresh``` method works just like the ```get``` method except that it returns a fresh instance even if the class that you are resolving is registered as a singlegon.

	$foo = $app->getFresh('bar');

> The ```getFresh``` method might not work as expected for classes registered using the ```registerInstance``` method.

--------------------------------------------------------

<a id="services"></a>

### Services

Services are an easy and clean way of registering classes in the IoC container. Mako registers a number of services in the container to make it easier for your to inject dependencies into your controllers and tasks.

The list of services that get registered is in the ```app/config/application.php``` configuration file. You can add your own or remove the ones that you don't need in your application.

The following services get registered in the container by default:

| Service                  | Class                              | Key              | Description                 | Required |
|--------------------------|------------------------------------|------------------|-----------------------------|----------|
|                          | mako\core\Application              | app              | Application / IoC container | ✔        |
| CacheService             | mako\cache\CacheManager            | cache            | Cache manager               | ✘        |
|                          | mako\core\Config                   | config           | Config loader               | ✔        |
| CryptoService            | mako\security\crypto\CryptoManager | crypto           | Crypto manager              | ✘        |
| DatabaseService          | mako\database\ConnectionManager    | database         | Database connection manager | ✘        |
| ErrorHandlerService      | mako\core\error\ErrorHandler       | errorhandler     | Error handler               | ✔        |
| GatekeeperService        | mako\auth\Gatekeeper               | gatekeeper       | Gatekeeper autentication    | ✘        |
| HumanizerService         | mako\utility\Humanizer             | humanizer        | Humanizer helper            | ✘        |
| I18nService              | mako\i18n\I18n                     | i18n             | Internationalization class  | ✘        |
| LoggerService            | Psr\Log\LoggerInterface            | logger           | Monolog logger              | ✔        |
| PaginationFactoryService | mako\pagination\PaginationFactory  | pagination       | Pagination factory          | ✘        |
| RedisService             | mako\redis\ConnectionManager       | redis            | Redis connection manager    | ✘        |
| RequestService           | mako\http\Request                  | request          | Request                     | ✔        |
| ResponseService          | mako\http\Response                 | response         | Response                    | ✔        |
| RouteService             | mako\http\routing\Routes           | routes           | Route collection            | ✔        |
| SessionService           | mako\session\Session               | session          | Session                     | ✘        |
| SignerService            | mako\security\Signer               | signer           | Signer                      | ✔        |
| URLBuilderService        | mako\http\routing\URLBuilder       | urlbuilder       | URL builder                 | ✘        |
| ValidatorFactoryService  | mako\validator\ValidatorFactory    | validatorfactory | Validation factory          | ✘        |
| ViewFactoryService       | mako\view\ViewFactory              | viewfactory      | View factory                | ✘        |

> Note that some of the services depend on each other (e.g. the session needs the database manager if you choose to store your sessions in the database).

--------------------------------------------------------

<a id="proxies"></a>

### Proxies

Mako 4 includes a set of proxy classes that you can use instead of injecting your dependencies through the controller and task constructors.

All examples in the documentation assume that you're using constructor injection but the syntax is the same (except for the static call).

	// Using constructor injection

	$connection = $this->database->connection();

	// Using the proxy

	$connection = Database::connection();

| Class                              | Proxy                              |
|------------------------------------|------------------------------------|
| mako\cache\CacheManager            | mako\proxies\Cache                 |
| mako\core\Config                   | mako\proxies\Config                |
| mako\security\crypto\CryptoManager | mako\proxies\Crypto                |
| mako\database\ConnectionManager    | mako\proxies\Database              |
| mako\auth\Gatekeeper               | mako\proxies\Gatekeeper            |
| mako\utility\Humanizer             | mako\proxies\Humanizer             |
| mako\i18n\I18n                     | mako\proxies\I18n                  |
| Psr\Log\LoggerInterface            | mako\proxies\Logger                |
| mako\pagination\PaginationFactory  | mako\proxies\Pagination            |
| mako\redis\ConnectionManager       | mako\proxies\Redis                 |
| mako\session\Session               | mako\proxies\Session               |
| mako\http\routing\URLBuilder       | mako\proxies\URL                   |
| mako\validator\ValidatorFactory    | mako\proxies\Validator             |
| mako\view\ViewFactory              | mako\proxies\View                  |