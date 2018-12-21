# Dependency injection

--------------------------------------------------------

* [Basics](#basics)
* [Contextual injection](#contextual_injection)
* [Replacing registered dependencies](#replacing_registered_dependencies)
* [Services](#services)
	- [Core](#services:core)
	- [Web](#services:web)
	- [CLI](#services:cli)
* [Container aware](#container_aware)

--------------------------------------------------------

Mako comes with an easy to use inversion of control container. Using dependency injection makes your application more maintainable and decoupled. Another benefit of using the dependency injection pattern is that it greatly increases the testability of the code, thus making it less prone to bugs.

> All controllers, migrations and commands are instantiated using the IoC container making it easy to inject dependencies.

--------------------------------------------------------

<a id="basics"></a>

### Basics

The `register` method allows you to register a dependency in the container.

```
$container->register(FooInterface::class, Foo::class);
```

It is possible to register a key along with the type hint. This will save a few keystrokes when resolving classes and also make it possible to access dependencies using overloading in [container aware](#container_aware) classes.

```
$container->register([FooInterface::class, 'foo'], Foo::class);
```

Additionally, the container allows you to register your dependencies using a closure.

```
$container->register([BarInterface::class, 'bar'], function($container)
{
	return new Bar('parameter value');
});
```

The `registerSingleton` method works just like the `register` method except that it makes sure that the same instance is returned every time the class is resolved through the container.

```
$container->registerSingleton([BarInterface::class, 'bar'], function($container)
{
	return new Bar('parameter value');
});
```

The `registerInstance` method is similar to the `registerSingleton` method. The only difference is that it allows you to register an existing instance in the container.

```
$container->registerInstance([BarInterface::class, 'bar'], new Bar('parameter value'));
```

The `has` method allows you to check for the presence of an item in the container.

```
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
```

The `get` method lets you resolve a dependency through the IoC container.

```
// Resolve the class using the type hint

$foo = $container->get(FooInterface::class);

// Or the optional key

$foo = $container->get('foo');
```

The class does not have to be registered in the container to be resolvable.

```
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
```

We can now resolve the `Depends` class using the IoC container. Both its dependencies will automatically be injected.

```
$depends = $container->get(Depends::class);
```

The `getFresh` method works just like the `get` method except that it returns a fresh instance even if the class that you are resolving is registered as a singleton.

```
$foo = $container->getFresh('bar');
```

> The `getFresh` method might not work as expected for classes registered using the `registerInstance` method.

The `call` method allows you to execute a callable and automatically inject its dependencies.

```
$returnValue = $container->call(function(\app\lib\FooInterface $foo, \app\lib\BarInterface $bar)
{
	// $foo and $bar will automatically be injected into the callable
});
```

--------------------------------------------------------

<a id="contextual_injection"></a>

### Contextual injection

Sometimes you'll need to inject different implementations of the same interface to different classes. This can easily be achieved with contextual dependency injection.

```
$container->registerContextualDependency(ClassA::class, FooBarInterface::class, FooBarImplementationA::class);
$container->registerContextualDependency(ClassB::class, FooBarInterface::class, FooBarImplementationB::class);
```

`ClassA` will now get the `FooBarImplementationA` implementation of the `FooBarInterface` while `ClassB` will get the `FooBarImplementationB` implementation.

--------------------------------------------------------

<a id="replacing_registered_dependencies"></a>

### Replacing registered dependencies

The container also allows you to replaces previously registered dependencies.

The `replace` method allows you to replace a previously registered dependency in the container.

```
$container->replace(FooInterface::class, OtherFoo::class);
```

The `replaceSingleton` method allows you to replace a previously registered singleton dependency in the container.

```
$container->replaceSingleton([BarInterface::class, 'bar'], function($container)
{
	return new OtherBar('parameter value');
});
```

The `replaceInstance` method allows you to replace a previously registered instance dependency in the container.

```
$container->replaceInstance([BarInterface::class, 'bar'], new OtherBar('parameter value'));
```

You can also replace instances that already been injected by the container thanks to the `onReplace` event.

In the following example we'll register an instance of the `Dependency` class along with a factory closure for the `Dependent` class. Inside the factory method we'll tell the container to replace the previous instance of the `Dependency` class using the `Dependent::replaceDependency()` method in the event that it gets replaced.

```
$container->registerInstance(Dependency::class, new Dependency('original'));

$container->register(Dependent::class, function($container)
{
	$dependent = new Dependent($container->get(Dependency::class));

	$container->onReplace(Dependency::class, [$dependent, 'replaceDependency']);

	return $dependent;
});

$dependent = $container->get(Dependent::class);

var_dump($dependent->dependency->value); // string(8) "original"

$container->replaceInstance(Dependency::class, new Dependency('replacement'));

var_dump($dependent->dependency->value); // string(11) "replacement"
```

In the example above we assumed that the `Dependent` class had a `replaceDependency` method. This might not always be the case so we can also use a closure to achieve the same result.

```
$container->onReplace(Dependency::class, (function($dependency)
{
	$this->dependency = $dependency;
})->bindTo($dependent, Dependent::class));
```

--------------------------------------------------------

<a id="services"></a>

### Services

Services are an easy and clean way of registering dependencies in the IoC container.

Mako includes a number of services for your convenience and you'll find a complete list in the `app/config/application.php` configuration file. You can add your own or remove the ones that you don't need in your application.

Services are split up in 3 groups. `Core` services are loaded in both web and cli environments while `web` and `cli` services are only loaded for web requests and command line operations respectively.

<a id="services:core"></a>

#### Core

| Service                  | Type hint                                  | Key              | Description                 | Required |
|--------------------------|--------------------------------------------|------------------|-----------------------------|----------|
|                          | mako\syringe\Container                     | container        | IoC container               | Yes      |
|                          | mako\application\Application               | app              | Application                 | Yes      |
|                          | mako\file\FileSystem                       | fileSystem       | File system abstraction     | Yes      |
|                          | mako\config\Config                         | config           | Config loader               | Yes      |
| CacheService             | mako\cache\CacheManager                    | cache            | Cache manager               | No       |
| CommandBusService        | mako\commander\CommandBusInterface         | commander        | Command bus                 | No       |
| CryptoService            | mako\security\crypto\CryptoManager         | crypto           | Crypto manager              | No       |
| DatabaseService          | mako\database\ConnectionManager            | database         | Database connection manager | No       |
| EventService             | mako\event\Event                           | event            | Event handler               | No       |
| GatekeeperService        | mako\auth\Gatekeeper                       | gatekeeper       | Gatekeeper authentication   | No       |
| HTTPService              | mako\http\Request                          | request          | Request                     | Yes      |
| HTTPService              | mako\http\Response                         | response         | Response                    | Yes      |
| HTTPService              | mako\http\routing\Routes                   | routes           | Route collection            | Yes      |
| HTTPService              | mako\http\routing\URLBuilder               | urlBuilder       | URL builder                 | Yes      |
| HumanizerService         | mako\utility\Humanizer                     | humanizer        | Humanizer helper            | No       |
| I18nService              | mako\i18n\I18n                             | i18n             | Internationalization class  | No       |
| LoggerService            | Psr\Log\LoggerInterface                    | logger           | Monolog logger              | No       |
| PaginationFactoryService | mako\pagination\PaginationFactoryInterface | pagination       | Pagination factory          | No       |
| RedisService             | mako\redis\ConnectionManager               | redis            | Redis connection manager    | No       |
| SessionService           | mako\session\Session                       | session          | Session                     | No       |
| SignerService            | mako\security\Signer                       | signer           | Signer                      | Yes      |
| ValidatorFactoryService  | mako\validator\ValidatorFactory            | validator        | Validation factory          | No       |
| ViewFactoryService       | mako\view\ViewFactory                      | view             | View factory                | No       |

<a id="services:web"></a>

#### Web

| Service             | Type hint                          | Key              | Description                 | Required |
|---------------------|------------------------------------|------------------|-----------------------------|----------|
| ErrorHandlerService | mako\error\ErrorHandler            | errorHandler     | Error handler               | No       |


<a id="services:cli"></a>

#### Cli

| Service                  | Type hint                          | Key              | Description                 | Required |
|--------------------------|------------------------------------|------------------|-----------------------------|----------|
|                          | mako\cli\input\Input               | input            | Input                       | Yes      |
|                          | mako\cli\output\Output             | output           | Output                      | Yes      |
| ErrorHandlerService      | mako\error\ErrorHandler            | errorHandler     | Error handler               | No       |



> Note that some of the services depend on each other (e.g. the session needs the database manager if you choose to store your sessions in the database).

--------------------------------------------------------

<a id="container_aware"></a>


### Container aware

You can also make a class that is instantiated by the container "container aware" by using the `ContainerAwareTrait`. This means that you can use the IoC container as a service locator if you prefer that.

> Note that controllers, migrations and tasks are container aware by default.

The IoC container is always available through the `$container` property.

```
$this->container->get('view');
```

The `ContainerAwareTrait` also implements the magic `__get()` method. This allows you to resolve classes through the IoC container using overloading.

```
$this->view; // Instance of mako\view\ViewFactory
```

> Note that resolving classes that are not registered as singletons in the container using overloading will result in a new instance every time. You should assign the resolved instance to a local variable if you need to perform multiple method calls on the object.
{.warning}
