# Upgrading

--------------------------------------------------------

* [Configuration](#configuration)
    - [Application](#configuration_application)
    - [Cache](#configuration_cache)
    - [Gatekeeper](#configuration_gatekeeper)
    - [Redis](#configuration_redis)
    - [Session](#configuration_session)
* [Cache](#cache)
    - [New return values](#cache_new_return_values)
* [Commands](#commands)
    - [Command descriptions](#commands_command_descriptions)
    - [Command arguments](#commands_command_arguments)
* [Database](#database)
    - [Dropped support for DB2](#database_dropped_support_for_db2)
    - [New return values](#database_new_return_values)
    - [Subquery changes](#database_subquery_changes)
    - [Set operation changes](#database_set_operation_changes)
* [Error handling](#error_handling)
* [Gatekeeper](#gatekeeper)
    - [New return values](#gatekeeper_new_return_values)
* [HTTP](#http)
    - [UploadedFile](#http_uploadedfile)
    - [Redirect](#http_redirect)
* [Validation](#validation)
* [Views](#views)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.3.x` to `7.0.x`.

--------------------------------------------------------

<a id="configuration"></a>

### Configuration

The undocumented cascading merging of configuration files has been removed from Mako 7 so you'll have to copy the entire configuration file if you want to define environment specific configuration files or override a package configuration value. 

<a id="configuration_application"></a>

#### Application

The new `logger` [configuration key](https://github.com/mako-framework/app/blob/master/app/config/application.php#L208-L226) must be added to the `application` configuration file.

```
/*
 * ---------------------------------------------------------
 * Logger
 * ---------------------------------------------------------
 *
 * channel: Log channel name
 * handler: Log handler(s) to use. The avaiable options out of the box are 'ErrorLog', 'Stream' and 'Syslog'.
 * syslog : Syslog specific options (https://linux.die.net/man/3/syslog).
 */
'logger' =>
[
	'channel' => 'mako',
	'handler' => ['Stream'],
	'syslog'  =>
	[
		'identifier' => 'Mako',
		'facility'   => LOG_USER,
	],
],
```

The `class_aliases` configuration key can be removed from the `application` configuration file since class aliasing has been removed.

<a id="configuration_cache"></a>

#### Cache

The `zenddisk` and `zendmemory` configuration keys can be removed from the `cache` configuration file as support for the two stores has been dropped.

<a id="configuration_gatekeeper"></a>

#### Gatekeeper

The `cookie_options.samesite` [configuration key](https://github.com/mako-framework/app/blob/master/app/config/gatekeeper.php#L109-L114) must be added to the `gatekeeper` configuration file.

```
/*
 * The supported values are 'Lax', 'Strict' and 'None'.
 *
 * @see https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
 */
'samesite' => 'Lax',
```

<a id="configuration_redis"></a>

#### Redis

The Redis client now supports RESP3 and the new Redis 6 ACL features. These new features can be leveraged using the new optional `username` and `resp` connection [configuration keys](https://github.com/mako-framework/app/blob/master/app/config/redis.php#L21-L30).

<a id="configuration_session"></a>

#### Session

The `cookie_options.samesite` [configuration key](https://github.com/mako-framework/app/blob/master/app/config/session.php#L73-L78) must be added to the `session` configuration file.

```
/*
 * The supported values are 'Lax', 'Strict' and 'None'.
 *
 * @see https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
 */
'samesite' => 'Lax',
```

--------------------------------------------------------

<a id="cache"></a>

### Cache

<a id="cache_new_return_values"></a>

#### New return values

The `StoreInterface::get()` method will now return `null` instead of `false` when an item does not exist in the cache.

--------------------------------------------------------

<a id="commands"></a>

### Commands

<a id="commands_command_descriptions"></a>

#### Command descriptions

Command descriptions can no longer be defined using the `$commandInformation` property and must be moved to the `$description` property instead.

```
protected $description = 'Command description';
```

Positional arguments and command options can no longer be defined using the `$commandInformation` property and must now be defined using the [getArguments](:base_url:/docs/:version:/command-line:commands#input:arguments-and-options) method.

```
public function getArguments(): array
{
    return
    [
        new Argument('--argument1', 'Description'),
        new Argument('--argument2', 'Description', Argument::IS_OPTIONAL),
    ];
}
```

--------------------------------------------------------


<a id="database"></a>

### Database

<a id="database_dropped_support_for_db2"></a>

#### Dropped support for DB2

Support for DB2 has been dropped. If you still want to use DB2 then you'll have to maintain your own DB2 connection and query builder classes.

<a id="database_new_return_values"></a>

#### New return values

The following methods in the database library will now return `null` instead of `false` when no matching record is found:

* `Connection::first()`
* `Connection::column()`
* `Query::first()`
* `Query::column()`
* `Query::get()`
* `ORM::get()`

<a id="database_subquery_changes"></a>

#### Subquery changes

Passing a `Closure` or `Query` instance to represent a subquery to the methods below no longer works. You must now pass a `Subquery` instance instead.

* `Query::table()`
* `Query::from()`
* `Query::into()`
* `Query::in()`
* `Query::notIn()`
* `Query::orIn()`
* `Query::orNotIn()`
* `Query::exists()`
* `Query::orExists()`
* `Query::notExists()`
* `Query::orNotExists()`
* `Query::with()`
* `Query::withRecursive()`

```
// Before:

$results = $query->table(function($query)
{
    $query->table('users');
})
->all();

// After:

$results = $query->table(new Subquery(function($query)
{
    $query->table('users');
}, 'users'))
->all();
```

<a id="database_set_operation_changes"></a>

#### Set operation changes

Passing a `Closure`, `Query` or `Subquery` instance to the following methods does no longer work and you must update your code to use the new and improved syntax introduced in Mako 6.2.

* `Query::union()`
* `Query::unionAll()`
* `Query::intersect()`
* `Query::intersectAll()`
* `Query::except()`
* `Query::exceptAll()`

```
// Before:

$combinedSales = $query->unionAll(function($query)
{
	$query->table('sales2015');
})
->table('sales2016')->all();

// After:

$combinedSales = $query->table('sales2015')->unionAll()->table('sales2016')->all();
```

--------------------------------------------------------

<a id="error_handling"></a>

### Error handling

The `ErrorHandler::disableLoggingFor()` method has been removed. Use the new `ErrorHandler::dontLog()` method or the `application.error_handler.dont_log` [configuration key](https://github.com/mako-framework/app/blob/master/app/config/application.php#L243-L246) instead.

--------------------------------------------------------

<a id="gatekeeper"></a>

### Gatekeeper

<a id="gatekeeper_new_return_values"></a>

#### New return values

The following methods in the Gatekeeper library will now return `null` instead of `false` when unable to retrieve data:

* `GroupRepositoryInterface::getByIdentifier()`
* `GroupRepository::getById()`
* `GroupRepository::getByName()`
* `UserRepositoryInterface::getByIdentifier()`
* `UserRepository::getByAccessToken()`
* `UserRepository::getByActionToken()`
* `UserRepository::getByEmail()`
* `UserRepository::getById()`
* `UserRepository::getByUsername()`

--------------------------------------------------------

<a id="http"></a>

### HTTP

<a id="http_uploadedfile"></a>

#### UploadedFile

The `UploadedFile::getName()` method has been renamed to `UploadedFile::getReportedFilename()` and the `UploadedFile::getReportedType()` method has been renamed to `UploadedFile::getReportedMimeType()`.

<a id="http_redirect"></a>

#### Redirect

The following `Redirect` methods have been removed:

* `Redirect::multipleChoices()`
* `Redirect::notModified()`
* `Redirect::notModified()`

--------------------------------------------------------

<a id="validation"></a>

### Validation

The following validation rule aliases have been removed:

* `max_filesize`
* `mimetype`

You should update your application to use the following rule names instead:

* `max_file_size`
* `mime_type`

--------------------------------------------------------

<a id="views"></a>

### Views

Support for the `{{$foo || 'Default'}}` and `{{$foo or 'Default'}}` template syntax has removed since it caused unexpected behaviour when printing the output of a ternary expression using the `||` and `or` operators. Templates must be updated to use the new `{{$foo, default: 'Default'}}` syntax instead.

The new `default` syntax also has a small change in behaviour. It will print out the value for variables containing `0`, `0.0` and `"0"`.

You can use the following regex to find your echo tags that still use the old syntax:

```
\{\{([^view:][^}]+)((\|\|){1}|\s+(or)\s+)([^}]+)\}\}
```
{.language-regex}