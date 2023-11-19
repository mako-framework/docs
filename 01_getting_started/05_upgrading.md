# Upgrading

--------------------------------------------------------

* [Application structure](#application_structure)
* [Routing](#routing)
    - [Constraints](#routing:constraints)
    - [Middleware](#routing:middleware)
* [Typed properties](#typed_properties)
    - [Built in middleware](#typed_properties:built_in_middleware)
    - [Commands](#typed_properties:commands)
    - [Custom validation rules](#typed_properties:custom_validation_rules)
    - [ORM](#typed_properties:orm)
    - [Packages](#typed_properties:packages)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `9.1.x` to `10.0.x`.

Mako `10.0` drops support for PHP version 8.0 and requires PHP version 8.1 or higher so make sure that your environment is up to date.

--------------------------------------------------------

### <a id="application_structure" href="#application_structure">Application structure</a>

Mako `10.0` comes with a new and improved directory strucute. Controllers and routing configuration has been moved to the `app\http` namespace. You can move the `app/controllers` and `app/routing` directories to the `app/http/(controllers|routing)` directory and update namespaces accordingly or you can continue to use the legacy structure. 

If you choose to keep using the legacy structure then you'll have to override the `HttpService`:

```
<?php

namespace app\services;

use mako\application\services\HTTPService as HttpBaseService;

class HttpService extends HttpBaseService
{
	protected function getRoutingPath(): string
	{
		return "{$this->app->getPath()}/routing";
	}
}
```

### <a id="routing" href="#routing">Routing</a>

#### <a id="routing:constraints" href="#routing:constraints">Constraints</a>

The `Router::registerConstraint()` and `Router::setConstraintAsGlobal()` methods have been removed and all calls to the methods must be removed from your `app/http/routing/constraints.php` configuration file.

If you want to register global constraints in Mako `10.0` then you'll have to use the new `Router::registerGlobalConstraint()` method:

```
<?php

// Before

$router->setConstraintAsGlobal([ApiVersionConstraint::class]);

// Now

$router->registerGlobalConstraint(ApiVersionConstraint::class);

// You can now pass arguments to global constraints

$router->registerGlobalConstraint(ApiVersionConstraint::class, version: '2.0');
```

Assigning constraints to a route is still done using the `Route::constraint()` method or `Constraint` attribute. However the `Route::constraint()` method will now have to be called multiple times when registering multiple constraints to a route:

```
<?php

// Before

$routes->get('/', fn () => 'Hello, world!')
->constraints([
    FooConstraint::class, 
    BarConstraint::class,
]);

// Now

$routes->get('/', fn () => 'Hello, world!')
->middleware(FooConstraint::class)
->middleware(BarConstraint::class);

// You can now pass arguments to constraints

$routes->get('/', fn () => 'Hello, world!')
->middleware(FooConstraint::class, foo: 123)
->middleware(BarConstraint::class, bar: 456);
```

Read all about the new constraint features [here](:base_url:/docs/:version:/routing-and-controllers:routing#route_constraints).

#### <a id="routing:middleware" href="#routing:middleware">Middleware</a>

The `Dispatcher::registerMiddleware()` and `Dispatcher::setMiddlewareAsGlobal()` methods have been removed and all calls to the methods must be removed from your `app/http/routing/middleware.php` configuration file.

If you want to register global middleware in Mako `10.0` then you'll have to use the new `Dispatcher::registerGlobalMiddleware()` method:

```
<?php

// Before

$dispatcher->setMiddlewareAsGlobal([AccessControl::class]);

// Now

$dispatcher->registerGlobalMiddleware(AccessControl::class);

// You can now pass arguments to global middleware

$dispatcher->registerGlobalMiddleware(CacheMiddleware::class, minutes: 60);
```

Setting middleware priority is still done using the `Dispatcher::setMiddlewarePriority()` method although instead of passing an array of middleware and priorities you make multiple calls to the method when registering multiple priorities:

```
<?php

// Before

$dispatcher->setMiddlewarePriority([
    FooMiddleware::class => 1,
    BarMiddleware::class => 2,
]);

// Now

$dispatcher->setMiddlewarePriority(FooMiddleware::class, 1);
$dispatcher->setMiddlewarePriority(BarMiddleware::class, 2);
```

Assigning middleware to a route is still done using the `Route::middleware()` method or `Middleware` attribute. However the `Route::middleware()` method will now have to be called multiple times when registering multiple middleware to a route:

```
// Before

$routes->get('/', fn () => 'Hello, world!')
->middleware([
    FooMiddleware::class, 
    BarMiddleware::class,
]);

// Now

$routes->get('/', fn () => 'Hello, world!')
->middleware(FooMiddleware::class)
->middleware(BarMiddleware::class);

// You can now pass arguments to middleware

$routes->get('/', fn () => 'Hello, world!')
->middleware(FooMiddleware::class, foo: 123)
->middleware(BarMiddleware::class, bar: 456);
```

The syntax for assigning middleware in route groups has also been sligtly changed:

```
<?php

// Without arguments

$options = [
	'middleware' => [CacheMiddleware::class],
];

$routes->group($options, function ($routes) {
	$routes->get('/', fn () => 'Hello, world!');
});

// With arguments

$options = [
	'middleware' => [[CacheMiddleware::class, ['minutes' => 60]]],
];

$routes->group($options, function ($routes) {
	$routes->get('/', fn () => 'Hello, world!');
});
```

Read all about the new middleware features [here](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware).

### <a id="typed_properties" href="#typed_properties">Typed properties</a>

Mako `10.0` introduces typed properties in all of its core classes so you'll have to update any class that extend the core.

#### <a id="typed_properties:built_in_middleware" href="#typed_properties:built_in_middleware">Built in middleware</a>

The `AccessControl` middleware properties must be updated to use typed properties:

```
<?php

namespace app\http\routing\middleware;

use mako\http\routing\middleware\AccessControl as AccessControlMiddleware;

class AccessControl extends AccessControlMiddleware
{
    protected bool $allowCredentials = false;

	protected bool $allowAllDomains = false;

	protected array $allowedDomains = [];

	protected array $allowedHeaders = [];

	protected array $allowedMethods = [];
}
```

The `ContentSecurityPolicy` middleware properties must be updated to use typed properties:

```
<?php

namespace app\http\routing\middleware;

use mako\http\routing\middleware\ContentSecurityPolicy as ContentSecurityPolicyMiddleware;

class ContentSecurityPolicy extends ContentSecurityPolicyMiddleware
{
	protected array $reportTo = [];

	protected bool $reportOnly = false;

	protected array $directives = [
		'base-uri'    => ['self'],
		'default-src' => ['self'],
		'object-src'  => ['none'],
	];
}
```

The `SecurityHeaders` middleware properties muse be updated to use typed properties:

```
<?php

namespace app\http\routing\middleware;

use mako\http\routing\middleware\SecurityHeaders as SecurityHeadersMiddleware;

class SecurityHeaders extends SecurityHeadersMiddleware
{
	protected array $headers = [
		'X-Content-Type-Options' => 'nosniff',
		'X-Frame-Options'        => 'sameorigin',
		'X-XSS-Protection'       => '1; mode=block',
	];
}
```

#### <a id="typed_properties:commands" href="#typed_properties:commands">Commands</a>

Your commands might not need any changes at all but here are all the properites that will have to be updated:

```
<?php

namespace app\console\commands;

use mako\reactor\Command;

class MyCommand extends Command
{
    protected string $command;

	protected string $description;
}
```

#### <a id="typed_properties:custom_validation_rules" href="#typed_properties:custom_validation_rules">Custom validation rules</a>

If you have implemented custom validation rules that use the `I18nAwareTrait` trait then you'll have to update the types of the `$i18nParameters` and `$i18nFieldNameParameters` properties.

```
<?php

namespace app\support\validation;

use mako\validator\rules\RuleInterface;
use mako\validator\rules\traits\I18nAwareTrait;

class MyRule implements RuleInterface
{
    use I18nAwareTrait;

    protected array $i18nParameters = [];
    protected array $i18nFieldNameParameters = [];

    // The rest of the validation rule implementation goes here ...
}
```

#### <a id="typed_properties:orm" href="#typed_properties:orm">ORM</a>

Your models might not need any changes at all but here are all the properites that will have to be updated:

```
<?php

namespace app\models;

use mako\database\midgard\ORM;

class Example extends ORM
{
    protected null|string $connectionName = null;

    protected string $tableName;

	protected string $foreignKeyName;

	protected string $primaryKey = 'id';

    protected int $primaryKeyType = ORM::PRIMARY_KEY_TYPE_INCREMENTING;

    protected array $including = [];

    protected array $cast = [];

    protected array $assignable = [];

	protected array $protected = [];

    protected string $dateOutputFormat = 'Y-m-d H:i:s';

    protected string $lockingColumn = 'lock_version'; // Used with the OptimisticLockingTrait trait

    protected array $nullable = []; // Used with the NullableTrait trait

    protected string $createdAtColumn = 'created_at'; // Used with the TimestampedTrait trait

    protected string $updatedAtColumn = 'updated_at'; // Used with the TimestampedTrait trait

    protected array $touch = []; // Used with the TimestampedTrait trait

    protected bool $shouldTouchOnInsert = true; // Used with the TimestampedTrait trait

    protected bool $shouldTouchOnUpdate = true; // Used with the TimestampedTrait trait

    protected bool $shouldTouchOnDelete = true; // Used with the TimestampedTrait trait
}
```

#### <a id="typed_properties:packages" href="#typed_properties:packages">Packages</a>

If you have created a custom Mako package then you'll have to update the package class to use typed properties:

```
<?php

namespace acme;

use mako\application\Package;

class MyPackage extends Package
{
    protected string $packageName = 'acme/my-package';

    protected array $commands = [];
}
```
