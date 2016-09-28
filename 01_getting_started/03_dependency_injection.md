# Dependency injection

--------------------------------------------------------

* [Basics](#basics)
* [Contextual injection](#contextual_injection)
* [Services](#services)
	- [Core](#services:core)
	- [Web](#services:web)
	- [CLI](#services:cli)
* [Container aware](#container_aware)

--------------------------------------------------------

Mako 4 comes with an easy to use inversion of control container. Using dependency injection makes your application more maintainable and decoupled. Another benefit of using the dependency injection pattern is that it greatly increases the testability of the code, thus making it less prone to bugs.

> All controllers, migrations and commands are instantiated using the IoC container making it easy to inject dependencies.

--------------------------------------------------------

<a id="basics"></a>

### Basics

The ```register``` method allows you to register a dependency in the container.

	$container->register(FooInterface::class, Foo::class);

You can also register a key along with the type hint to so that you can save a few keystrokes when resolving classes.

	$container->register([FooInterface::class, 'foo'], Foo::class);

You can also register your dependencies using a closure. The closure will not be executed before it is required.

	$container->register([BarInterface::class, 'bar'], function($container)
	{
		return new Bar('parameter value');
	});

The ```registerSingleton``` method does the same as the ```register``` method except that it makes sure that the same instance is returned every time the class is resolved through the container.

	$container->registerSingleton([BarInterface::class, 'bar'], function($container)
	{
		return new Bar('parameter value');
	});

The ```registerInstance``` method is similar to the ```registerSingleton``` method. The only difference is that it allows you to register an existing instance in the container.

	$container->registerInstance([BarInterface::class, 'bar'], new Bar('parameter value'));

The ```has``` method allows you to check for the presence of an item in the container.

	// Check using the type hint

	if($container->has(BarInterface::class))
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

	$foo = $container->get(FooInterface::class);

	// Or the optional key

	$foo = $container->get('foo');

The class does not have to be registered in the container to be resolvable.

	<?php

	class Depends
	{
		protected $foo;
		protected $bar;

		public function __construct(FooInterface $foo, BarInterface $bar)
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

The ```call``` method allows you to execute a callable and automatically inject its dependencies.

	$returnValue = $container->call(function(\app\lib\FooInterface $foo, \app\lib\BarInterface $bar)
	{
		// $foo and $bar will automatically be injected into the callable
	});

--------------------------------------------------------

<a id="contextual_injection"></a>

### Contextual injection

Sometimes you'll need to inject different implementations of the same interface to different classes. This can easily be achieved with contextual dependency injection.

	$container->registerContextualDependency(ClassA::class, FooBarInterface::class, FooBarImplementationA::class);
	$container->registerContextualDependency(ClassB::class, FooBarInterface::class, FooBarImplementationB::class);

```ClassA``` will now get the ```FooBarImplementationA``` implementation of the ```FooBarInterface``` while ```ClassB``` will get the ```FooBarImplementationB``` implementation.

--------------------------------------------------------

<a id="services"></a>

### Services

Services are an easy and clean way of registering dependecies in the IoC container.

Mako includes a number of services for your convenience and you'll find a complete list in the ```app/config/application.php``` configuration file. You can add your own or remove the ones that you don't need in your application.

Services are split up in 3 groups. ```Core``` services are loaded both web and cli environments while ```web``` and ```cli``` services are only loaded for web requests and command line operations respectively.

<a id="services:core"></a>

#### Core

| Service                  | Type hint                                  | Key              | Description                 | Required |
|--------------------------|--------------------------------------------|------------------|-----------------------------|----------|
|                          | mako\syringe\Syringe                       | container        | IoC container               | ✔        |
|                          | mako\application\Application               | app              | Application                 | ✔        |
|                          | mako\file\FileSystem                       | fileSystem       | File system abstraction     | ✔        |
|                          | mako\config\Config                         | config           | Config loader               | ✔        |
| CacheService             | mako\cache\CacheManager                    | cache            | Cache manager               | ✘        |
| CommandBusService        | mako\commander\CommandBusInterface         | commander        | Command bus                 | ✘        |
| CryptoService            | mako\security\crypto\CryptoManager         | crypto           | Crypto manager              | ✘        |
| DatabaseService          | mako\database\ConnectionManager            | database         | Database connection manager | ✘        |
| EventService             | mako\event\Event                           | event            | Event handler               | ✘        |
| GatekeeperService        | mako\auth\Gatekeeper                       | gatekeeper       | Gatekeeper autentication    | ✘        |
| HTTPService              | mako\http\Request                          | request          | Request                     | ✔        |
| HTTPService              | mako\http\Response                         | response         | Response                    | ✔        |
| HTTPService              | mako\http\routing\Routes                   | routes           | Route collection            | ✔        |
| HTTPService              | mako\http\routing\URLBuilder               | urlBuilder       | URL builder                 | ✔        |
| HumanizerService         | mako\utility\Humanizer                     | humanizer        | Humanizer helper            | ✘        |
| I18nService              | mako\i18n\I18n                             | i18n             | Internationalization class  | ✘        |
| LoggerService            | Psr\Log\LoggerInterface                    | logger           | Monolog logger              | ✘        |
| PaginationFactoryService | mako\pagination\PaginationFactoryInterface | pagination       | Pagination factory          | ✘        |
| RedisService             | mako\redis\ConnectionManager               | redis            | Redis connection manager    | ✘        |
| SessionService           | mako\session\Session                       | session          | Session                     | ✘        |
| SignerService            | mako\security\Signer                       | signer           | Signer                      | ✔        |
| ValidatorFactoryService  | mako\validator\ValidatorFactory            | validator        | Validation factory          | ✘        |
| ViewFactoryService       | mako\view\ViewFactory                      | view             | View factory                | ✘        |

<a id="services:web"></a>

#### Web

| Service                  | Type hint                          | Key              | Description                 | Required |
|--------------------------|------------------------------------|------------------|-----------------------------|----------|
| ErrorHandlerService      | mako\error\ErrorHandler            | errorHandler     | Error handler               | ✘        |


<a id="services:cli"></a>

#### Cli

| Service                  | Type hint                          | Key              | Description                 | Required |
|--------------------------|------------------------------------|------------------|-----------------------------|----------|
| InputService             | mako\cli\input\Input               | input            | Input                       | ✔        |
| OutputService            | mako\cli\output\Ouput              | output           | Output                      | ✔        |
| ErrorHandlerService      | mako\error\ErrorHandler            | errorHandler     | Error handler               | ✘        |



> Note that some of the services depend on each other (e.g. the session needs the database manager if you choose to store your sessions in the database).

--------------------------------------------------------

<a id="container_aware"></a>


### Container aware

You can also make a class that is instantiated by the container "container aware" by using the ```ContainerAwareTrait```. This means that you can use the IoC container as a service locator if you prefer that.

> Note that controllers, migrations and tasks are container aware by default.

The IoC container is always avaiable through the ```$container``` property.

	$this->container->get('view');

The ```ContainerAwareTrait``` also implements the magic ```__get()``` method. This allows you to resolve classes through the IoC container using overloading.

	$this->view; // Instance of mako\view\ViewFactory