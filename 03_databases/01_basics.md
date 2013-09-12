# Databases

--------------------------------------------------------

* [Basics](#basics)
* [Query builder](#query_builder)
* [Transactions](#transactions)
* [Accessing PDO](#accessing_pdo)

--------------------------------------------------------

The database class provides a simple way of handling database connections in your application.

--------------------------------------------------------

<a id="basics"></a>

### Basics

Creating a database connection is done using the ```connection``` method.

	// Returns connection object using the "default" database configuration defined in the config file

	$connection = Database::connection();

	// Returns connection object using the "mydb" database configuration defined in the config file

	$connection = Database::connection('mydb');

The ```query``` method lests you execute a query and it will returns data for ```SELECT``` queries, number of rows affected for ```DELETE``` and ```UPDATE``` queries and a boolean value for other queries.

There is a third optional parameter that lets you set the fetch method. The different fetch modes are ```FETCH_ALL``` (default), ```FETCH_FIRST``` and ```FETCH_COLUMN```.

	$connection->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', array('fruit', 'banana'));

	// You can also use the following syntax to perform a query against the default connection

	Database::query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', array('fruit', 'banana'));

	// There's also a handy syntax for assigning arrays for use in "IN" clauses

	Database::query('SELECT * FROM `foo` WHERE `bar` IN ([?])', array(array('banana', 'apple')));

The ```all``` method executes a query and returns an array containing all of the result set rows.

	$rows = $connection->all('SELECT * from `foo` WHERE `bar` = ?', array($bar));

	// You can also use the following syntax to perform a query against the default connection

	$rows = Database::all('SELECT * from `foo` WHERE `bar` = ?', array($bar));

The ```first``` method executes a query and returns the first row of the result set.

	$row = $connection->first('SELECT * from `foo` WHERE `bar` = ?', array($bar));

	// You can also use the following syntax to perform a query against the default connection

	$row = Database::first('SELECT * from `foo` WHERE `bar` = ?', array($bar));

The ```column``` method executes a query and returns the value of the first column of the first row of the result set.

	$email = $connection->column('SELECT `email` from `users` WHERE `id` = ?', array(1));

	// You can also use the following syntax to perform a query against the default connection

	$email = Database::column('SELECT `email` from `users` WHERE `id` = ?', array(1));

--------------------------------------------------------

<a id="query_builder"></a>

### Query builder

The ```table``` method returns an instance of the [query builder](:base_url:/docs/:version:/databases:query-builder).

	$rows = $connection->table('foo')->where('bar', '=', $bar)->all();

	// You can also use the following syntax to perform a query against the default connection

	$rows = Database::table('foo')->where('bar', '=', $bar)->all();

--------------------------------------------------------

<a id="transactions"></a>

### Transactions

The ```transaction``` method provides a handy shortcut for performing database transactions. It returns TRUE if the transaction was successful and FALSE if it failed.

> Transactions only work if the storage engine you're using supports them.

	$success = $connection->transaction(function($connection)
	{
		$connection->table('accounts')->where('user_id', '=', 10)->decrement('cash', 100);

		$connection->table('accounts')->where('user_id', '=', 20)->increment('cash', 100);
	});

	// You can also use the following syntax to perform a query against the default connection

	$success = Database::transaction(function($connection)
	{
		$connection->table('accounts')->where('user_id', '=', 10)->decrement('cash', 100);

		$connection->table('accounts')->where('user_id', '=', 20)->increment('cash', 100);
	});

--------------------------------------------------------

<a id="accessing_pdo"></a>

### Accessing PDO

You can also access the [PDO](http://php.net/manual/en/book.pdo.php) object directly when needed.

	try
	{
		$connection->pdo->beginTransaction();
		
		$connection->query('DROP TABLE `foo`');

		$connection->query('INSERT INTO `foo` (`bar`, `baz`) VALUES (?, ?)', array('fruit', 'banana'));

		$connection->pdo->commit();
	}
	catch(PDOException $e)
	{
		$connection->pdo->rollBack();
	}