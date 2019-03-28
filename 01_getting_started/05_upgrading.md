# Upgrading

--------------------------------------------------------

* [Application config](#application_configuration)
* [Authentication](#authentication)
* [Database](#database)
	- [ORM](#database:orm)
* [FileSystem](#filesystem)
* [HTTP](#http)
	- [Request](#http:request)
		- [Headers](#http:request:headers)
	- [Response](#http:response)
		- [JSON](#http:response:json)
		- [Redirect](#http:response:redirect)
	- [Exceptions](#http:exceptions)
	- [Routing](#http:routing)
		- [Constraints](#http:routing:constraints)
		- [Middleware](#http:routing:middleware)
* [Migrations](#migrations)
* [Packages](#packages)
* [Services](#services)
* [Validator](#validator)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `5.7.x` to `6.0.x`. You should also take a look at the changelog for a full list of changes to internal classes and methods.

--------------------------------------------------------

<a id="application_configuration"></a>

### Application configuration

The `base_url` key of the `application` config file should be set to `null` instead of an empty string if the application is using auto-detection.

--------------------------------------------------------

<a id="authentication"></a>

### Authentication

The `mako\gatekeeper\Authentication` class has been renamed to `mako\gatekeeper\Gatekeeper`. Any type hints or uses of the class constants must be updated to use the new class name.

The [`policies`](https://github.com/mako-framework/app/blob/d28f95e4f2744af1e12f651b7d7b27e065f143aa/app/config/gatekeeper.php#L38-L49) key must be added to the `app/config/gatekeeper.php` config file.

--------------------------------------------------------

<a id="database"></a>

### Database

<a id="database:orm"></a>

#### ORM

Support for magic scope methods has been removed. Use the [`scope`](:base_url:/docs/:version:/databases-sql:orm#scopes) method instead.

--------------------------------------------------------

<a id="filesystem"></a>

### FileSystem

The following methods that were deprecated in Mako 5.7.0 have been removed:

* `FileSystem::mime()`
* `FileSystem::hash()`
* `FileSystem::hmac()`

The same functionality (and more) can now be found in the [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) class.

--------------------------------------------------------

<a id="http"></a>

### HTTP

<a id="http:request"></a>

#### Request

Several of the `mako\http\Request` methods have been renamed for consistency.

| Before         | Now               |
|----------------|-------------------|
| basePath       | getBasePath       |
| baseURL        | getBaseURL        |
| contentType    | getContentType    |
| ip             | getIp             |
| language       | getLanguage       |
| languagePrefix | getLanguagePrefix |
| method         | getMethod         |
| password       | getPassword       |
| path           | getPath           |
| realMethod     | getRealMethod     |
| referer        | getReferrer       |
| scriptName     | getScriptName     |
| username       | getUsername       |

<a id="http:request:headers"></a>

##### Headers

Several of the `mako\http\request\Headers` methods have been renamed for consistency.

| Before                 | Now                       |
|------------------------|---------------------------|
| acceptableContentTypes | getAcceptableContentTypes |
| acceptableLanguages    | getAcceptableLanguages    |
| acceptableCharsets     | getAcceptableCharsets     |
| acceptableEncodings    | getAcceptableEncodings    |

<a id="http:response"></a>

#### Response

Several of the `mako\http\Response` methods have been renamed for consistency.

| Before       | Now              |
|-------------|-------------------|
| body        | setBody           |
| cache       | enableCaching     |
| charset     | setCharset        |
| compression | enableCompression |
| status      | setStatus         |
| type        | setType           |

<a id="http:response:json"></a>

##### JSON

The `status` method of the `JSON` response builder has been renamed to `setStatus`.

<a id="http:response:redirect"></a>

##### Redirect

The `status` method of the `Redirect` response sender has been renamed to `setStatus`.

<a id="http:exceptions"></a>

#### Exceptions

The `RequestException` has been renamed to `HttpException`.

<a id="http:routing"></a>

#### Routing

<a id="http:routing:constraints"></a>

##### Constraints

Constraint parameters are now injected through the constructor and the `Constraint` base class has been removed. Constraints should now implement the `ConstraintInterface`. Check out the [documentation](:base_url:/docs/:version:/routing-and-controllers:routing#route_constraints) for more details.

<a id="http:routing:middleware"></a>

##### Middleware

Middleware parameters are now injected through the constructor and the `Middleware` base class has been removed. Middleware should now implement the `MiddlewareInterface`. Check out the [documentation](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware) for more details.

--------------------------------------------------------

<a id="migrations"></a>

### Migrations

The return type of the `up` and `down` methods of your [migrations](:base_url:/docs/:version:/databases-sql:migrations) must be `void`.

```
/**
 * Makes changes to the database structure.
 */
public function up(): void
{

}

/**
 * Reverts the database changes.
 */
public function down(): void
{

}
```

--------------------------------------------------------

<a id="packages"></a>

### Packages

If any of your [packages](:base_url:/docs/:version:/packages:packages) override the `Package:boostrap()` method then you'll have to make sure that the return type is `void`.

```
/**
 * {@inheritdoc}
 */
protected function bootstrap(): void
{

}
```

--------------------------------------------------------

<a id="services"></a>

### Services

The return type of the `register` method of your custom [services](:base_url:/docs/:version:/getting-started:dependency-injection#services) must be `void`.

```
/**
 * {@inheritdoc}
 */
public function register(): void
{

}
```

--------------------------------------------------------

<a id="validator"></a>

### Validator

Custom validation rules that take parameters must no longer implement the `WithParametersInterface`. Parameters will have to be injected through the constructor.
