# Databases

--------------------------------------------------------

* [Connections](#connections)
	- [Basics](#connections:basics)
	- [Transactions](#transactions)
	- [Magic shortcut](#connections:magic_shortcut)
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

	$connection = $this->database->connection();

	// Returns connection object using the "mydb" database configuration defined in the config file

	$connection = $this->database->connection('mydb');

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

The ```Connection::queryAndCound``` method will return the number of rows modified by the query.

	$count = $connection->queryAndCound('UPDATE `users` SET `email` = ?', ['foo@example.org']);

	$count = $connection->queryAndCound('DELETE FROM `users`');

<a id="transactions"></a>

#### Transactions

The ```Connection::transaction``` method provides a handy shortcut for performing database transactions. It returns the return value of the transaction closure.

> Transactions only work if the storage engine you're using supports them.

	$success = $connection->transaction(function($connection)
	{
		$connection->builder()->table('accounts')->where('user_id', '=', 10)->decrement('cash', 100);

		$connection->builder()->table('accounts')->where('user_id', '=', 20)->increment('cash', 100);
	});

<a id="connections:magic_shortcut"></a>

#### Magic shortcut

You can access the default database connection directly without having to go through the ```connection``` method thanks to the magic ```__call``` method.

	$this->database->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', ['fruit', 'banana']);

--------------------------------------------------------

<a id="query_builder"></a>

### Query builder

The ```Connection::builder``` method returns an instance of the [query builder](:base_url:/docs/:version:/databases:query-builder).

	$rows = $connection->builder()->table('foo')->where('bar', '=', $bar)->all();

--------------------------------------------------------

<a id="accessing_the_underlying_pdo_instance"></a>

### Accessing the underlying PDO instance

You can also access the [PDO](http://php.net/manual/en/book.pdo.php) object directly when needed.

	$serverVersion = $connection->getPDO()->getAttribute(PDO::ATTR_SERVER_VERSION);