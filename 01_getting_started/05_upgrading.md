# Upgrading

--------------------------------------------------------

* [FileSystem](#filesystem)
* [HTTP](#http)
* [Migrations](#migrations)
* [Packages](#packages)
* [Services](#services)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `5.7.x` to `6.0.x`.

--------------------------------------------------------

<a id="filesystem"></a>

### FileSystem

The following methods that were deprecated in Mako 5.7.0 have been removed:

* `FileSystem::mime()`
* `FileSystem::hash()`
* `FileSystem::hmac()`

The same functionality (and more) can now be found in the [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) class.

<a id="http"></a>

### HTTP

The `RequestException` has been renamed to `HttpException`.

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
