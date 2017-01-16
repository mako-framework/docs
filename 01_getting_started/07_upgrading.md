# Upgrading

--------------------------------------------------------

* [Application](#application)
	- [Configuration](#application:configuration)
* [Framework](#framework)
	- [Middleware](#framework:middleware)
	- [ORM foreign keys](#framework:orm-foreign-keys)
	- [Query builder](#framework:query-builder)
	- [Validation](#framework:validation)
	- [Views](#framework:views)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```5.0.x``` to ```5.1.x```.

--------------------------------------------------------

<a id="application"></a>

### Application

<a id="application:configuration"></a>

#### Configuration

Add the new ```base_url``` config key to your application [configuration file](https://github.com/mako-framework/app/blob/5.1/app/config/application.php#L16).

<a id="framework"></a>

### Framework

<a id="framework:middleware"></a>

#### Middleware

There is a [new syntax](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware) for passing arguments to route middleware rules.

<a id="framework:orm-foreign-keys"></a>

#### ORM foreign-keys

The ORM now uses ```Str::camel2underscored()``` instead of ```strtolower()``` when generating the foreign key names. Most applications should be unaffected but you can configure the foreign key name using the ```$foreignKeyName``` property so that you don't have to do any changes to your database.

<a id="framework:validation"></a>

#### Validation

There is a [new syntax](:base_url:/docs/:version:/learn-more:validation) for passing arguments to validation rules.

Piped validation rules (deprecated since Mako 3.6) are no longer supported, use an array instead.

<a id="framework:views"></a>

#### Views

Views are no longer auto rendered by the response class. They should be rendered in the controller.
