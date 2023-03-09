# Databases

--------------------------------------------------------

* [Connections](#connections)
	- [Basics](#connections:basics)
	- [Connection status](#connection_status)
	- [Magic shortcut](#connections:magic_shortcut)
* [Transactions](#transactions)
	- [Basics](#transactions:basics)
	- [Savepoints](#transactions:savepoints)
* [Query builder](#query_builder)
* [Accessing the underlying PDO instance](#accessing_the_underlying_pdo_instance)

--------------------------------------------------------

The database connection manager provides a simple way of handling database connections in your application.

--------------------------------------------------------

### <a id="connections" href="#connections">Connections</a>

#### <a id="connections:basics" href="#connections:basics">Basics</a>

Creating a database connection is done using the `ConnectionManager::getConnection()` method.

```
// Returns connection object using the "default" database configuration defined in the config file

$connection = $this->database->getConnection();

// Returns connection object using the "mydb" database configuration defined in the config file

$connection = $this->database->getConnection('mydb');
```

The `Connection::query()` method lets you execute a query. It returns `true` on success and `false` on failure.

```
$connection->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', ['fruit', 'banana']);
```

The `Connection::first()` method executes a query and returns the first row of the result set or `null` if nothing is found.

```
$row = $connection->first('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);
```

The `Connection::firstOrThrow()` method executes a query and returns the first row of the result set or throws an exception if nothing is found.

```
// By default if throws a mako\database\exceptions\NotFoundException

$row = $connection->firstOrThrow('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);

// But you can make it throw any exception you want 
// You can for example throw a mako\http\exceptions\NotFoundException to display a 404 page

$row = $connection->firstOrThrow('SELECT * FROM `foo` WHERE `bar` = ?', [$bar], NotFoundException::class);
```

The `Connection::all()` method executes a query and returns an array containing all of the result set rows.

```
$rows = $connection->all('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);

// There's also a handy syntax for assigning arrays for use in "IN" clauses

$rows = $connection->all('SELECT * FROM `foo` WHERE `bar` IN ([?])', [['banana', 'apple']]);
```

The `Connection::yield()` method executes a query and returns a generator that lets you iterate over the result set rows. This is very useful if you want to process a large dataset without having to worry about memory consumption.

```
$rows = $connection->yield('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);
```

> Note that when using MySQL you might have to configure PDO to use [unbuffered queries](https://php.net/manual/en/mysqlinfo.concepts.buffering.php) for this to work as expected.
{.warning}

The `Connection::column()` method executes a query and returns the value of the first column of the first row of the result set or `null` if nothing is found.

```
$email = $connection->column('SELECT `email` FROM `users` WHERE `id` = ?', [1]);
```

The `Connection::columns()` method executes a query and returns an array containing the values of the first column.

```
$email = $connection->columns('SELECT `email` FROM `users`');
```

The `Connection::pairs()` method will return an array where the first column is used as array keys and the second column is used as array values.

```
$pairs = $connection->pairs('SELECT `id`, `email` FROM `users`');
```

The `Connection::queryAndCount()` method will return the number of rows modified by the query.

```
$count = $connection->queryAndCount('UPDATE `users` SET `email` = ?', ['foo@example.org']);

$count = $connection->queryAndCount('DELETE FROM `users`');
```

#### <a id="connection_status" href="#connection_status">Connection status</a>

You can check if a connection is still alive using the `Connection::isAlive()` method. It will return `true` if it is and `false` if not.

```
$isConnectionAlive = $connection->isAlive();
```

You can attempt to reconnect using the `Connection::reconnect()` method.

```
$connection->reconnect();
```

> You can configure the connection to automatically reconnect in the `app/config/database.php` configuration file. Note that Mako will not attempt to automatically reconnect if the connection was lost during a transaction.

#### <a id="connections:magic_shortcut" href="#connections:magic_shortcut">Magic shortcut</a>

You can access the default database connection directly without having to go through the `connection` method thanks to the magic `__call` method.

```
$this->database->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', ['fruit', 'banana']);
```

--------------------------------------------------------

### <a id="transactions" href="#transactions">Transactions</a>

#### <a id="transactions:basics" href="#transactions:basics">Basics</a>

> Transactions only work if the storage engine you're using supports them.

You begin a transaction using the `Connection::beginTransaction()` method.

```
$connection->beginTransaction();
```

Committing the transaction is done using the `Connection::commitTransaction()` method.

```
$connection->commitTransaction();
```

Rolling back the transaction is done using the `Connection::rollBackTransaction()` method.

```
$connection->rollBackTransaction();
```

You can check whether or not you're already in a transaction using the `Connection::inTransaction()` method.

```
$inTransaction = $connection->inTransaction();
```

The `Connection::transaction()` method provides a handy shortcut for performing simple database transactions. Any failed queries in the closure will automatically roll back the transaction.

```
$connection->transaction(function ($connection)
{
	$connection->getQuery()->table('accounts')->where('user_id', '=', 10)->decrement('cash', 100);

	$connection->getQuery()->table('accounts')->where('user_id', '=', 20)->increment('cash', 100);
});
```

#### <a id="transactions:savepoints" href="#transactions:savepoints">Savepoints</a>

Nested transactions are also supported using savepoints.

In the example below we'll create a user named `foo`. The nested transaction that would have created a second user named `bar` will fail since the table name is misspelled.

The parent transaction is unaffected and `foo` user is still created. If you want your entire transaction to roll back when the nested transaction fails then you can just re-throw the exception.

```
try
{
	$connection->beginTransaction();

	$connection->getQuery()->table('users')->insert(['username' => 'foo']);

	{
		$connection->beginTransaction();

		try
		{
			$connection->getQuery()->table('usesr')->insert(['username' => 'bar']);

			$connection->commitTransaction();
		}
		catch(PDOException $e)
		{
				$connection->rollbackTransaction();

				// throw $e; // Re-throw the exception to roll back the parent transaction as well
		}
	}

	$connection->commitTransaction();
}
catch(PDOException $e)
{
		$connection->rollbackTransaction();
}
```

You can get the transaction nesting level at any point using the `Connection::getTransactionNestingLevel()` method.

```
$nestingLevel = $connection->getTransactionNestingLevel();
```

Transaction nesting is also possible when using the `Connection::transaction()` method but keep in mind that the entire transaction will be rolled back if any of the nested transactions fail.

--------------------------------------------------------

### <a id="query_builder" href="#query_builder">Query builder</a>

The `Connection::getQuery()` method returns an instance of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder).

```
$rows = $connection->getQuery()->table('foo')->where('bar', '=', $bar)->all();
```

--------------------------------------------------------

### <a id="accessing_the_underlying_pdo_instance" href="#accessing_the_underlying_pdo_instance">Accessing the underlying PDO instance</a>

You can also access the [PDO](https://php.net/manual/en/book.pdo.php) object directly when needed.

```
$serverVersion = $connection->getPDO()->getAttribute(PDO::ATTR_SERVER_VERSION);
```
