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

<a id="connections"></a>

### Connections

<a id="connections:basics"></a>

#### Basics

Creating a database connection is done using the ```ConnectionManager::connection``` method.

	// Returns connection object using the "default" database configuration defined in the config file

	$connection = $connection->connection();

	// Returns connection object using the "mydb" database configuration defined in the config file

	$connection = $connection->connection('mydb');

The ```Connection::query``` method lets you execute a query. It returns ```TRUE``` on success and ```FALSE``` on failure.

	$connection->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', ['fruit', 'banana']);

The ```Connection::all``` method executes a query and returns an array containing all of the result set rows.

	$rows = $connection->all('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);

	// There's also a handy syntax for assigning arrays for use in "IN" clauses

	$rows = $connection->all('SELECT * FROM `foo` WHERE `bar` IN ([?])', [['banana', 'apple']]);

The ```Connection::first``` method executes a query and returns the first row of the result set.

	$row = $connection->first('SELECT * FROM `foo` WHERE `bar` = ?', [$bar]);

The ```Connection::column``` method executes a query and returns the value of the first column of the first row of the result set.

	$email = $connection->column('SELECT `email` FROM `users` WHERE `id` = ?', [1]);

The ```Connection::columns``` method executes a query and returns an array containing the values of the first column.

	$email = $connection->columns('SELECT `email` FROM `users`');

The ```Connection::queryAndCount``` method will return the number of rows modified by the query.

	$count = $connection->queryAndCount('UPDATE `users` SET `email` = ?', ['foo@example.org']);

	$count = $connection->queryAndCount('DELETE FROM `users`');

<a id="connection_status"></a>

#### Connection status

You can check if a connection is still alive using the ```Connection::isAlive()``` method. It will return TRUE if it is and FALSE if not.

	$isConnectionAlive = $connection->isAlive();

You can attempt to reconnect using the ```Connection::reconnect()``` method.

	$connection->reconnect();

> You can configure the connection to automatically reconnect in the ```app/config/database.php``` configuration file. Note that Mako will not attempt to automatically reconnect if the connection was lost during a transaction.

<a id="connections:magic_shortcut"></a>

#### Magic shortcut

You can access the default database connection directly without having to go through the ```connection``` method thanks to the magic ```__call``` method.

	$connection->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', ['fruit', 'banana']);

--------------------------------------------------------

<a id="transactions"></a>

### Transactions


<a id="transactions:basics"></a>

#### Basics

> Transactions only work if the storage engine you're using supports them.

You begin a transaction using the ```Connection::beginTransaction``` method.

	$connection->beginTransaction();

Committing the transaction is done using the ```Connection::commitTransaction``` method.

	$connection->commitTransaction();

Rolling back the transaction is done using the ```Connection::rollBackTransaction``` method.

	$connection->rollBackTransaction();

You can check whether or not you're already in a transaction using the ```Connection::inTransaction``` method.

	$inTransaction = $connection->inTransaction();

The ```Connection::transaction``` method provides a handy shortcut for performing simple database transactions. Any failed queries in the closure will automatically roll back the transaction.

	$connection->transaction(function($connection)
	{
		$connection->builder()->table('accounts')->where('user_id', '=', 10)->decrement('cash', 100);

		$connection->builder()->table('accounts')->where('user_id', '=', 20)->increment('cash', 100);
	});

<a id="transactions:savepoints"></a>

#### Savepoints

Nested transactions are also supported using savepoints.

In the example below we'll decrease the cash total of user `1` by `100` and increase the cash total of user `2` by `100`. The nested transaction that would have increased the cash total of user `1` by another `1000` fails and is rolled back since the table name is misspelled.

The parent transaction is unafected and the transfer between user `1` and `2` is still committed. If you want your entire transaction to roll back when the nested transaction fails then you can just re-throw the exception.

	try
	{
		$connection->beginTransaction();

		$connection->builder()->table('accounts')->where('id', '=', 1)->decrement('cash', 100);

		$connection->builder()->table('accounts')->where('id', '=', 2)->increment('cash', 100);

		{
			$connection->beginTransaction();

			try
			{
				$connection->builder()->table('accountss')->where('id', '=', 2)->increment('cash', 1000);

				$connection->commitTransaction();
			}
			catch(PDOException $e)
			{
					$connection->rollbackTransaction();
			}
		}

		$connection->commitTransaction();
	}
	catch(PDOException $e)
	{
			$connection->rollbackTransaction();
	}

Transaction nesting is also possible when using the ```Connection::transaction``` method but keep in mind that the entire transaction will be rolled back if any of the nested transactions fail.

You can get the transaction nesting level at any point using the ```Connection::getTransactionNestingLevel``` method.

	$nestingLevel = $connection->getTransactionNestingLevel();

--------------------------------------------------------

<a id="query_builder"></a>

### Query builder

The ```Connection::builder()``` method returns an instance of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder).

	$rows = $connection->builder()->table('foo')->where('bar', '=', $bar)->all();

You can also use the ```Connection::table()``` convenience method if you want to skip the call to the ```Connection::builder()``` method.

	$rows = $connection->table('foo')->where('bar', '=', $bar)->all();

--------------------------------------------------------

<a id="accessing_the_underlying_pdo_instance"></a>

### Accessing the underlying PDO instance

You can also access the [PDO](http://php.net/manual/en/book.pdo.php) object directly when needed.

	$serverVersion = $connection->getPDO()->getAttribute(PDO::ATTR_SERVER_VERSION);