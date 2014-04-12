# Dependency injection

--------------------------------------------------------

* [Basics](#basics)
* [Services](#services)

--------------------------------------------------------

Mako 4 comes with a inversion of controler container that allows you to inject dependencies into your controllers and command line tasks. Using dependency injection makes your application more maintainable and decoupled. Another benefit of using the dependency injection pattern is that it greatly increases the testability of the code, thus making it less prone to bugs.

--------------------------------------------------------

<a id="basics"></a>

### Basics

The IoC container is by default avaiable in the ```app/bootstrap.php``` and ```app/routes.php``` files.

#### Registering dependencies

The ```register``` method allows you to register a dependency in the container.

	$container->register('app\lib\FooInterface', 'app\lib\Foo');

You can also register a key along with the type hint to so that you can save a few keystrokes when resolving classes.

	$container->register(['app\lib\FooInterface', 'foo'], 'app\lib\Foo');

You can also register your dependencies using a closure. The closure will not be executed before it is required.

	$container->register(['app\lib\BarInterface', 'bar'], function($container)
	{
		return new \app\lib\Bar('parameter value');
	});

The ```registerSingleton``` method does the same as the ```register``` method except that it makes sure that the same instance is returned every time the class is resolved through the container.

	$container->registerSingleton(['app\lib\BarInterface', 'bar'], function($container)
	{
		return new \app\lib\Bar('parameter value');
	});

The ```registerInstance``` method is similar to the ```registerSingleton``` method. The only difference is that it allows you to register an existing instance in the container.

	$container->registerInstance(['app\lib\BarInterface', 'bar'], new \app\lib\Bar('parameter value'));

The ```has``` method allows you to check for the presence of an item in the container.

	// Check using the type hint

	if($container->has('app\lib\BarInterface'))
	{
		// do something
	}
	
	// Or the optional key

	if($container->has('bar'))
	{
		// do something
	}

The ```get``` method lets you resolve a dependency through the IoC container.

	// Resolve the class using the type hint

	$foo = $container->get('app\lib\FooInterface');

	// Or the optional key

	$foo = $container->get('foo');

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

	$depends = $container->get('Depends');

The ```getFresh``` method works just like the ```get``` method except that it returns a fresh instance even if the class that you are resolving is registered as a singlegon.

	$foo = $container->getFresh('bar');

> The ```getFresh``` method might not work as expected for classes registered using the ```registerInstance``` method.

--------------------------------------------------------

<a id="services"></a>

### Services

Services are an easy and clean way of registering dependecies in the IoC container. 

Mako includes a number of services for your convenience and you'll find a complete list in the ```app/config/application.php``` configuration file. You can add your own or remove the ones that you don't need in your application.

| Service                  | Class                              | Key              | Description                 | Required |
|--------------------------|------------------------------------|------------------|-----------------------------|----------|
|                          | mako\syringe\Syringe               | container        | IoC container               | ✔        |
|                          | mako\core\Application              | app              | Application                 | ✔        |
|                          | mako\core\Config                   | config           | Config loader               | ✔        |
| CacheService             | mako\cache\CacheManager            | cache            | Cache manager               | ✘        |
| CryptoService            | mako\security\crypto\CryptoManager | crypto           | Crypto manager              | ✘        |
| DatabaseService          | mako\database\ConnectionManager    | database         | Database connection manager | ✘        |
| ErrorHandlerService      | mako\core\error\ErrorHandler       | errorhandler     | Error handler               | ✔        |
| GatekeeperService        | mako\auth\Gatekeeper               | gatekeeper       | Gatekeeper autentication    | ✘        |
| HumanizerService         | mako\utility\Humanizer             | humanizer        | Humanizer helper            | ✘        |
| I18nService              | mako\i18n\I18n                     | i18n             | Internationalization class  | ✘        |
| LoggerService            | Psr\Log\LoggerInterface            | logger           | Monolog logger              | ✔        |
| PaginationFactoryService | mako\pagination\PaginationFactory  | pagination       | Pagination factory          | ✘        |
| RedisService             | mako\redis\ConnectionManager       | redis            | Redis connection manager    | ✘        |
| RequestService           | mako\http\Request                  | request          | Request                     | ✔        |
| ResponseService          | mako\http\Response                 | response         | Response                    | ✔        |
| RouteService             | mako\http\routing\Routes           | routes           | Route collection            | ✔        |
| SessionService           | mako\session\Session               | session          | Session                     | ✘        |
| SignerService            | mako\security\Signer               | signer           | Signer                      | ✔        |
| URLBuilderService        | mako\http\routing\URLBuilder       | urlbuilder       | URL builder                 | ✘        |
| ValidatorFactoryService  | mako\validator\ValidatorFactory    | validator        | Validation factory          | ✘        |
| ViewFactoryService       | mako\view\ViewFactory              | view             | View factory                | ✘        |

> Note that some of the services depend on each other (e.g. the session needs the database manager if you choose to store your sessions in the database).