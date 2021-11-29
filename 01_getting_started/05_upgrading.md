# Upgrading

--------------------------------------------------------

* [Cache](#cache)
* [Commands](#commands)
* [Crypto](#crypto)
* [Database](#database)
    - [Connection](#database:connection)
    - [ORM](#database:orm)
* [Routing](#routing)
    - [Access Control middleware](#routing:access-control-middleware)
* [Validation](#validation)
    - [Custom rules](#validation:custom-rules)
    - [Renamed rules](#validation:renamed-rules)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `7.3.x` to `8.0.x`.

Mako `8.0` drops support for PHP version `7.3` and requires PHP version `7.4` or higher so make sure that your web server is up to date.

--------------------------------------------------------

<a id="cache"></a>

### Cache

The deprecated `CacheManager::instance()` method has been removed. Use the `CacheManager::getInstance()` or `CacheManager::getStore()` methods instead.

```
// Before

$cache = $this->cache->instance();

// Now

$cache = $this->cache->getInstance();

// Or

$cache = $this->cache->getStore();
```

--------------------------------------------------------

<a id="commands"></a>

### Commands

The following reactor commands have been renamed:

| Before                 | Now                    |
|------------------------|------------------------|
| app.generate_preloader | app:generate-preloader |
| app.generate_secret    | app.generate_secret    |
| app.generate-key       | app:generate-key       |
| app.routes             | app:routes             |
| cache.clear            | cache:clear            |
| cache.remove           | cache:remove           |
| migrate.create         | migration:create       |
| migrate.down           | migration:down         |
| migrate.reset          | migration:reset        |
| migrate.status         | migration:status       |
| migrate.up             | migration:up           |
| server                 | app:server             |

--------------------------------------------------------

<a id="crypto"></a>

### Crypto

The deprecated `CryptoManager::instance()` method has been removed. Use the `CryptoManager::getInstance()` or `CryptoManager::getEncrypter()` methods instead.

```
// Before

$cache = $this->crypto->instance();

// Now

$cache = $this->crypto->getInstance();

// Or

$cache = $this->crypto->getEncrypter();
```

--------------------------------------------------------

<a id="database"></a>

### Database

<a id="database:connection"></a>

#### Connection

The deprecated `Connection::builder()` method has been removed. Use the `Connection::getQuery()` method instead.

```
// Before

$query = $connection->builder();

// Now

$query = $connection->getQuery();
```

The deprecated `Connection::table()` method has been removed. It does not have direct replacement.

```
// Before

$query = $connection->table('foobar');

// Now

$query = $connection->getQuery()->table('foobar');
```

<a id="database:orm"></a>

#### ORM

The deprecated `ORM::builder()` method has been removed. Use the `ORM::getQuery()` method instead.

```
// Before

$query = (new ORM)->builder();

// Now

$query = (new ORM)->getQuery();
```

--------------------------------------------------------

<a id="routing"></a>

### Routing

The deprecated `Route::namespace()` has been removed along with the deprecated `namespace` route group option. Route class actions should now be registered as arrays instead of strings.

```
<?php

// Before

$routes->group(['namespace' => 'app\controllers'], function($routes)
{
    $routes->get('/', 'Index::welcome');
});
```

```
<?php

// Now

use app\controllers\Index;

$routes->get('/', [Index::class, 'welcome']);
```

<a id="routing:access-control-middleware"></a>

#### Access Control middleware

The deprecated `AccessControlAllowOrigin` has been removed and replaced by the new and improved `AccessControl` middleware.

--------------------------------------------------------

<a id="validation"></a>

### Validation

<a id="validation:custom-rules"></a>

#### Custom rules

The `RuleInterface::validate()` method now has `3` parameters instead of `2`.

```
// Before

public function validate($value, array $input): bool;

// Now

public function validate($value, string $field, array $input): bool;
```

<a id="validation:renamed-rules"></a>

#### Renamed rules

The following input validation rules have been renamed:

| Before           | Now                      |
|------------------|--------------------------|
| float            | numeric:float            |
| integer          | numeric:int              |
| natural_non_zero | numeric:natural_non_zero |
| natural          | numeric:natural          |
