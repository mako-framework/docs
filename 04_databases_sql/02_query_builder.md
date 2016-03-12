# Query builder

--------------------------------------------------------

* [Getting a query builder instance](#getting_a_query_builder_instance)
* [Fetching data](#fetching_data)
* [Inserting data](#inserting_data)
* [Updating data](#updating_data)
* [Deleting data](#deleting_data)
* [Aggregates](#aggregates)
* [WHERE clauses](#where_clauses)
* [WHERE BETWEEN clauses](#where_between_clauses)
* [WHERE IN clauses](#where_in_clauses)
* [WHERE IS NULL clauses](#where_is_null_clauses)
* [WHERE EXISTS clauses](#where_exists_clauses)
* [JOIN clauses](#join_clauses)
* [GROUP BY clauses](#group_by_clauses)
* [HAVING clauses](#having_clauses)
* [ORDER BY clauses](#order_by_clauses)
* [LIMIT and OFFSET clauses](#limit_and_offset_clauses)
* [Row-level locking](#row_level_locking)

--------------------------------------------------------

The query builder allows you to programmatically build SQL queries.

The query builder currently supports the following dialects:

* DB2
* Firebird
* MariaDB
* MySQL
* NuoDB
* Oracle
* PostgreSQL
* SQLite
* SQLServer

All queries executed by the query builder use prepared statements, thus mitigating the risk of SQL injections.

--------------------------------------------------------

<a id="getting_a_query_builder_instance"></a>

### Getting a query builder instance

You can create a query builder instance using the ```Connection::builder()``` method.

	$query = $connection->builder();

You can also skip the call to the ```Connection::builder()``` method using the ```Connection::table()``` method.

	$query = $conncetion->table('foobar');

--------------------------------------------------------

<a id="fetching_data"></a>

### Fetching data

Fetching all rows is done using the ```all``` method.

	$persons = $query->table('persons')->all();

	// You can also specify which columns you want to include in the result set

	$persons = $query->table('persons')->select(['name', 'email'])->all();

	// To make a distinct selection use the distinct method

	$persons = $query->table('persons')->select(['name', 'email'])->distinct()->all();

> Note that the ```all``` method returns a result set. So you'll need to use the ```isEmpty``` method to check if it's empty.

Selecting from the results of a subquery is also possible.

	$persons = $query->table(function($query)
	{
		$query->table('persons')->select(['name'])->distinct();
	})
	->where('name', '!=', 'John Doe')
	->all();

You can also use the Subquery class instead of a closure if you need a specific table alias.

	$persons = $query->table
	(
		new Subquery(function($query)
		{
			$query->table('persons')->select(['name'])->distinct();
		}, 'distinct_names')
	)
	->where('name', '!=', 'John Doe')
	->all();

Advanced column selections can also be made using raw SQL and subqueries.

	$persons = $query->table('persons')->select
	(
		[
			'name',
			'email',
			new Raw("CASE gender WHEN 'm' THEN 'male' ELSE 'female' END AS gender"),
			new Subquery(function($query)
			{
				$query->table('persons')->select([new Raw('AVG(age)']));
			}, 'average_age')
		]
	)->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

If you only want to retrieve a single row you can use the ```first``` method.

	$person = $query->table('persons')->where('id', '=', 1)->first();

Fetching the value of a single column is done using the ```column``` method.

	$email = $query->table('persons')->select(['email'])->where('id', '=', 1)->column();

	// You can also use the following syntax

	$email = $query->table('persons')->where('id', '=', 1)->column('email');

It is also possible to fetch an array containing the values of a single column using the ```columns``` method.

	 $emails = $query->table('persons')->select(['email'])->columns();

	 // You can also use the following syntax

	 $emails = $query->table('persons')->columns('email');

If you need to process large ammounts of data then the ```batch``` method will help you limit the memory usage of your application. The default batch size is a 1000 records but you can override this using the optional second parameter.

You can also set the offset starting point and offset end point using the optional third and fourth parameters respectively. This is useful if you have parallel workers processing data.

	$query->table('persons')->ascending('id')->batch(function($batch)
	{
		// Process the batch here
	});

--------------------------------------------------------

<a id="inserting_data"></a>

### Inserting data

Inserting data is done using the ```insert``` method.

	$query->table('foobars')->insert(['field1' => 'foo', 'field2' => new DateTime()]);

You can also insert data using the ```insertAndGetId``` method. It will create the record and return the generated auto increment id.

	$query->table('foobars')->insertAndGetId(['field1' => 'foo', 'field2' => new DateTime()]);

> When working with [PostgreSQL](http://www.postgresql.org) the ```insertAndGetId``` method assumes that the sequence follows the default naming convention (```<table_name>_<primary_key_name>_seq```) You can override the default primary key name (```id```) by using the optional second parameter.

--------------------------------------------------------

<a id="updating_data"></a>

### Updating data

Updating data is done using the ```update``` method.

	$query->table('foobars')
	->where('id', '=', 10)
	->update(['field1' => 'foo', 'field2' => 'bar', 'field3' => time()]);

There are also shortcuts for incrementing and decrementing column values:

	$query->table('articles')->where('id', '=', 1)->increment('views');

	$query->table('articles')->where('id', '=', 1)->increment('views', 10);

	$query->table('shows')->where('id', '=', 1)->decrement('tickets')

	$query->table('shows')->where('id', '=', 1)->decrement('tickets', 50);

--------------------------------------------------------

<a id="deleting_data"></a>

### Deleting data

Deleting data is done using the ```delete``` method.

	$query->table('articles')->where('id', '=', 10)->delete();

--------------------------------------------------------

<a id="aggregates"></a>

### Aggregates

The query builder also includes a few handy shortcuts to the most common aggregate functions:

	// Counting

	$count = $query->table('persons')->count();

	$count = $query->table('persons')->where('age', '>', 25)->count();

	// Average value

	$height = $query->table('persons')->avg('height');

	$height = $query->table('persons')->where('age', '>', 25)->avg('height');

	// Largest value

	$height = $query->table('persons')->max('height');

	$height = $query->table('persons')->where('age', '>', 25)->max('height');

	// Smallest value

	$height = $query->table('persons')->min('height');

	$height = $query->table('persons')->where('age', '>', 25)->min('height');

	// Sum

	$height = $query->table('persons')->sum('height');

	$height = $query->table('persons')->where('age', '>', 25)->sum('height');

--------------------------------------------------------

<a id="where_clauses"></a>

### WHERE clauses

where(), whereRaw(), orWhere(), orWhereRaw()

	// SELECT * FROM `persons` WHERE `age` > 25

	$persons = $query->table('persons')->where('age', '>', 25)->all();

	// SELECT * FROM `persons` WHERE `age` > 25 OR `age` < 20

	$persons = $query->table('persons')->where('age', '>', 25)->orWhere('age', '<', 20)->all();

	// SELECT * FROM `persons` WHERE (`age` > 25 AND `height` > 180) AND `email` IS NOT NULL

	$persons = $query->table('persons')
	->where(function($query)
	{
		$query->where('age', '>', 25);
		$query->where('height', '>', 180);
	})
	->isNotNull('email')
	->all();

You can also use the eq(), notEq(), lt(), lte(), gt(), gte(), like(), notLike(), orEq(), orNotEq(), orLt(), orLte(), orGt(), orGte(), orLike(), orNotLike(), eqRaw(), notEqRaw(), ltRaw(), lteRaw(), gtRaw(), gteRaw(), likeRaw(), notLikeRaw(), orEqRaw(), orNotEqRaw(), orLtRaw(), orLteRaw(), orGtRaw(), orGteRaw(), orLikeRaw(), orNotLikeRaw() convenience methods.

	// SELECT * FROM `persons` WHERE `age` > 25

	$persons = $query->table('persons')->gt('age', 25)->all();

	// SELECT * FROM `persons` WHERE `age` > 25 OR `age` < 20

	$persons = $query->table('persons')->gt('age', 25)->orLt('age', 20)->all();

	// SELECT * FROM `persons` WHERE (`age` > 25 AND `height` > 180) AND `email` IS NOT NULL

	$persons = $query->table('persons')
	->where(function($query)
	{
		$query->gt('age', 25);
		$query->gt('height', 180);
	})
	->isNotNull('email')
	->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

<a id="where_between_clauses"></a>

### WHERE BETWEEN clauses

between(), orBetween(), notBetween(), orNotBetween()

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25

	$persons = $query->table('persons')->between('age', 20, 25)->all();

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25 OR `age` BETWEEN 30 AND 35

	$persons = $query->table('persons')->between('age', 20, 25)->orBetween('age', 30, 35)->all();

<a id="where_in_clauses"></a>

### WHERE IN clauses

in(), orIn(), notIn(), orNotIn()

	// SELECT * FROM `persons` WHERE `id` IN (1, 2, 3, 4, 5)

	$persons = $query->table('persons')->in('id', [1, 2, 3, 4, 5])->all();

	// SELECT * FROM `persons` WHERE `id` IN (SELECT `id` FROM `persons` WHERE `id` != 1)

	$persons = $query->table('persons')
	->in('id', function($query)
	{
		$query->table('persons')->select(['id'])->where('id', '!=', 1);
	})
	->all();

<a id="where_is_null_clauses"></a>

### WHERE IS NULL clauses

isNull(), orIsNull(), isNotNull(), orIsNotNull()

	// SELECT * FROM `persons` WHERE `address` IS NULL

	$persons = $query->table('persons')->isNull('address')->all();

<a id="where_exists_clauses"></a>

### WHERE EXISTS clauses

exists(), orExists(), notExists(), orNotExists()

	// SELECT * FROM `persons` WHERE EXISTS (SELECT * FROM `cars` WHERE `cars`.`person_id` = `persons`.`id`)

	$persons = $query->table('persons')
	->exists(function($query)
	{
		$query->table('cars')->whereRaw('cars.person_id', '=', 'persons.id');
	})
	->all();

<a id="join_clauses"></a>

### JOIN clauses

join(), joinRaw(), leftJoin(), leftJoinRaw()

	// SELECT * FROM `persons` INNER JOIN `phones` ON `persons`.`id` = `phones`.`user_id`

	$persons = $query->table('persons')->join('phones', 'persons.id', '=', 'phones.user_id')->all();

	// SELECT * FROM `persons` AS `u` INNER JOIN `phones` AS `p` ON
	// `u`.`id` = `p`.`user_id` OR `u`.`phone_number` = `p`.`number`

	$persons = $query->table('persons as u')
	->join('phones as p', function($join)
	{
		$join->on('u.id', '=', 'p.user_id');
		$join->orOn('u.phone_number', '=', 'p.number');
	})
	->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

<a id="group_by_clauses"></a>

### GROUP BY clauses

groupBy()

	// SELECT `customer`, `order_date`, SUM(`order_price`) as `sum` FROM `orders` GROUP BY `customer`

	$customers = $query->table('orders')
	->select(['customer', new Raw('SUM(price) as sum')])
	->groupBy('customer')
	->all();

	// SELECT `customer`, `order_date`, SUM(`order_price`) as `sum` FROM `orders` GROUP BY `customer`, `order_date`

	$customers = $query->table('orders')
	->select(['customer', 'order_date', new Raw('SUM(price) as sum')])
	->groupBy(['customer', 'order_date'])
	->all();

<a id="having_clauses"></a>

### HAVING clauses

having(), havingRaw(), orHaving(), orHavingRaw()

	// SELECT `customer`, SUM(`price`) AS `sum FROM `orders` GROUP BY `customer` HAVING SUM(`price`) < 2000

	$customers = $query->table('orders')
	->select(['customer', new Raw('SUM(price) as sum')])
	->groupBy('customer')
	->havingRaw('SUM(price)', '<', 2000)
	->all();

<a id="order_by_clauses"></a>

### ORDER BY clauses

orderBy(), orderByRaw(), descending(), descendingRaw(), ascending(), ascendingRaw()

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $query->table('persons')->orderBy('name', 'asc')->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $query->table('persons')->ascending('name')->all();

	// SELECT * FROM `persons` ORDER BY `name` DESC

	$persons = $query->table('persons')->descending('name')->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC, `age` DESC

	$persons = $query->table('persons')->orderBy('name', 'asc')->orderBy('age', 'desc')->all();

	// SELECT * FROM `persons` ORDER BY `name`, `age` ASC

	$persons = $query->table('persons')->orderBy(['name', 'age'], 'asc')->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

<a id="limit_and_offset_clauses"></a>

### LIMIT and OFFSET clauses

limit(), offset(), paginate()

	// SELECT * FROM `persons` LIMIT 10

	$persons = $query->table('persons')->limit(10)->all();

	// SELECT * FROM `persons` LIMIT 10 OFFSET 10

	$persons = $query->table('persons')->limit(10)->offset(10)->all();

You can also use the [```paginate``` method](:base_url:/docs/:version:/learn-more:pagination#usage_with_the_query_builder) to limit your results.

	// SELECT * FROM `persons` LIMIT 10 OFFSET 0

	$persons = $query->table('persons')->paginate(10);

<a id="row_level_locking"></a>

### Row-level locking

The `lock()` method can be used to enable row-level locking during database transactions.

	// SELECT * FROM `persons` WHERE `age` =  30 FOR UPDATE

	$persons = $query->table('persons')->where('age', '=', 30)->lock()->all();

It will use an exclusive lock by default but you can enable shared locking by passing ```FALSE``` to the ```lock()``` method.

	// SELECT * FROM `persons` WHERE `age` =  30 LOCK IN SHARE MODE

	$persons = $query->table('persons')->where('age', '=', 30)->lock(false)->all();

It is also possible to provide a custom locking clause.

	// SELECT * FROM `persons` WHERE `age` =  30 CUSTOM LOCK

	$persons = $query->table('persons')->where('age', '=', 30)->lock('CUSTOM LOCK')->all();

Here's an overview of the locking clauses generated for the different RDBMSes that support row-level locking.

| RDBMS      | Exclusive lock          | Shared lock                 |
|------------|-------------------------|-----------------------------|
| DB2        | FOR UPDATE WITH RS      | FOR READ ONLY WITH RS       |
| Firebird   | FOR UPDATE WITH LOCK    | WITH LOCK                   |
| MySQL      | FOR UPDATE              | LOCK IN SHARE MODE          |
| NuoDB      | FOR UPDATE              | LOCK IN SHARE MODE          |
| Oracle     | FOR UPDATE              | FOR UPDATE                  |
| PostgreSQL | FOR UPDATE              | FOR SHARE                   |
| SQLServer  | WITH (UPDLOCK, ROWLOCK) | WITH (HOLDLOCK, ROWLOCK)    |

> Row-level locking will gracefully degrade for any RDBMS that doesn't support the feature.
