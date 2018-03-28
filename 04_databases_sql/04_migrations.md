# Migrations

--------------------------------------------------------

* [Usage](#usage)
	- [Enabling migrations](#usage:enabling_migrations)
	- [Creating migrations](#usage:creating_migrations)
	- [Running migrations](#usage:running_migrations)
	- [Rolling back migrations](#usage:rolling_back_migrations)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

Database migrations offer a convenient way to alter your databases in a structured and organized manner. They allow you to commit and roll back schema changes.

Mako migrations are created and executed from the [reactor CLI tool](:base_url:/docs/:version:/command-line:basics).

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:enabling_migrations"></a>

#### Enabling migrations

You need to add a table called `mako_migrations` to your database. The table will be used to keep track of the migrations that have been ran. Here's the SQL for creating the table using MySQL:

```
CREATE TABLE `mako_migrations`
(
	`batch` int(10) unsigned NOT NULL,
	`package` varchar(255) DEFAULT NULL,
	`version` varchar(255) NOT NULL
);
```
{.language-sql}

<a id="usage:creating_migrations"></a>

#### Creating migrations

Creating a migration is done from the reactor CLI tool:

```
// Creates an application migration

php reactor migrate.create

// Creates a migration for the "foobar" package

php reactor migrate.create --package="vendor/package"

// Creates a migration with a description

php reactor migrate.create --description="Creates session table"
```

Running the `create` commands will return the following messages:

```
Migration created at "/var/www/app/migrations/Migration_20140824100019.php".

Migration created at "/var/www/app/packages/foobar/migrations/Migration_20140824100317.php".

Migration created at "/var/www/app/migrations/Migration_20140824100019.php".
```

The generated migration will contain a skeleton class with two methods, `up` and `down`. The database connection manager is available in both methods using the `$this->database` property.

```
<?php

class Migration_20120824100019 extends Migration
{
	/**
	 * Makes changes to the database structure.
	 *
	 * @access  public
	 */

	public function up()
	{

	}

	/**
	 * Reverts the database changes.
	 *
	 * @access  public
	 */

	public function down()
	{

	}
}
```

<a id="usage:running_migrations"></a>

#### Running migrations

You can check if there are any outstanding migrations using the `status` command:

```
php reactor migrate.status
```
{.language-none}

If there are outstanding migrations then you can run them like this:

```
php reactor migrate.up
```
{.language-none}

This will show you the names of the migrations that were executed:

```
Ran the following migrations:

+-------------------------------------------+-----------------------+
| Migration                                 | Description           |
+-------------------------------------------+-----------------------+
| Migration_20140824100019                  |                       |
| Migration_20140824100317 (vendor/package) |                       |
| Migration_20140824100019                  | Creates session table |
+-------------------------------------------+-----------------------+
```
{.language-none}

<a id="usage:rolling_back_migrations"></a>

#### Rolling back migrations

If you need to revert the changes made to your database then you can use the `down` command. This will roll back the last batch of migrations executed.

```
php reactor migrate.down
```
{.language-none}

This will show you the migrations that were rolled back:

```
Rolled back the following migrations:

+-------------------------------------------+-----------------------+
| Migration                                 | Description           |
+-------------------------------------------+-----------------------+
| Migration_20140824100019                  | Creates session table |
| Migration_20140824100317 (vendor/package) |                       |
| Migration_20140824100019                  |                       |
+-------------------------------------------+-----------------------+
```
{.language-none}

You can roll back multiple batches by telling the rollback command how many batches you want to roll back.

```
php reactor migrate.down 2
```
{.language-none}

If you want to roll back all database changes in one go then you can use the `reset` command.

```
php reactor migrate.reset
```
{.language-none}

This will prompt you for confirmation. To force the reset just use the `force` option.

```
php reactor migrate.reset --force
```
{.language-none}

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Migrations are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

```
<?php

class Migration_20120824100019 extends Migration
{
	protected $config;

	public function __construct(ConnectionManager $connectionManager, Config $config)
	{
		parent::__construct($connectionManager);

		$this->config = $config;
	}
}
```

> Note that migrations expect the first constructor parameter to be an instance of the `ConnectionManager` class.


You can also inject dependencies directly into the up and down methods since they are executed by the `Container::call()` method.

```
public function down(LoggerInterface $log)
{
	$log->info('Executed the down method of the ' . static::class . ' migration');
}
```

Migrations are also `container aware`. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).
