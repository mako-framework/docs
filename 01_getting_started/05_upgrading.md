# Upgrading

--------------------------------------------------------

* [Application](#application)
	- [Migrations](#application:migrations)
	- [Services](#application:services)
* [Packages](#packages)
* [Framework](#framework)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `5.7.x` to `6.0.x`.

--------------------------------------------------------

<a id="application"></a>

### Application

<a id="application:migrations"></a>

#### Migrations

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

> This requirement also applies for package migrations.

<a id="application:services"></a>

#### Services

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

<a id="framework"></a>

### Framework
