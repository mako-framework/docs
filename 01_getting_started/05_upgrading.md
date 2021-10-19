# Upgrading

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `7.2.x` to `7.3.x`. 

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

#### Access control middleware

The `AccessControlAllowOrigin` middleware is deprecated and will be removed in Mako 8.0. It is replaced by the new `AccessControl` middleware which can set all the `Access-Control-*` headers.

--------------------------------------------------------

#### Cache

The `CacheManager::instance()` method has been deprecated and replaced by the `CacheManager::getInstance()` method. You can also use the `CacheManager::getStore()` alias method.

```
// Before

$cache = $this->cache->instance();

// After

$cache = $this->cache->getInstance();

// Or

$cache = $this->cache->getStore();
```

--------------------------------------------------------

#### Databases

The `ConnectionManager::connection()` method has been deprecated and is replaced by the `ConnectionManager::getConnection()` method.

```
// Before

$connection = $this->database->connection();

// After

$connection = $this->database->getConnection();
```

The `Connection::builder()` method has been deprecated and replaced by the `Connection::getQuery()` method.

```
// Before

$query = $connection->builder();

// After

$query = $connection->getQuery();
```

The `Connection::table()` method has been deprecated without a replacement.

```
// Before

$query = $connection->table('foobar');

// After

$query = $connection->getQuery()->table('foobar');
```

The `ORM::builder()` method has been deprecated and replaced by the `ORM::getQuery()` method.

```
// Before

$query = (new Model)->builder();

// After

$query = (new Model)->getQuery();
```

--------------------------------------------------------

#### Encryption

The `CryptoManager::instance()` method has been deprecated and replaced by the `CryptoManager::getInstance()` method. You can also use the `CryptoManager::getEncrypter()` alias method.

```
// Before

$encrypter = $this->crypto->instance();

// After

$encrypter = $this->crypto->getInstance();

// Or

$encrypter = $this->crypto->getEncrypter();
```

--------------------------------------------------------

#### Routing

The `Route::namespace()` method (and group config key) as well as the old way of registering class method actions as strings has been deprecated. Routes should now be registered using the `*::class` constant.

```
<?php

$routes->group(['namespace' => 'app\controllers'], function($routes)
{
    $routes->get('/', 'Index::welcome');
});
```
The code above should now be written like this:
```
<?php

use app\controllers\Index;

$routes->get('/', [Index::class, 'welcome']);
```