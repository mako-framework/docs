# Query builder

--------------------------------------------------------

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

<a id="fetching_data"></a>

### Fetching data

Fetching all rows is done using the ```Query::all``` method.

	$persons = $connection->builder()->table('persons')->all();

	// You can also specify which columns you want to include in the result set

	$persons = $connection->builder()->table('persons')->all(['name', 'email']);

	// To make a distinct selection use the distinct method

	$persons = $connection->builder()->table('persons')->distinct()->all(['name', 'email']);

Selecting from the results of a subquery is also possible.

	// Using a closure

	$persons = $connection->builder->table(function($query)
	{
		$query->table('persons')->distinct()->columns(['name']);
	})
	->where('name', '!=', 'John Doe')
	->all();

	// You can also use the Subquery class if you need a specific table alias

	$persons = $connection->builder()->table
	(
		new Subquery
		(
			$connection->builder()->table('persons')->distinct()->columns(['name']), 'distinct_names'
		)
	)
	->where('name', '!=', 'John Doe')
	->all();

You can also make advanced column selections with raw SQL and subqueries

	$persons = $connection->builder()->table('persons')->all(array
	(
		'name',
		'email',
		new Raw("CASE gender WHEN 'm' THEN 'male' ELSE 'female' END AS gender"),
		new Subquery
		(
			$connection->builder()->table('persons')->columns([new Raw('AVG(age)'])), 'average_age'
		)
	));

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

If you only want to retrieve a single row you can use the ```Query::first``` method.

	$person = $connection->builder()->table('persons')->where('id', '=', 1)->first();

	// You can also specify which columns you want to include in the result set

	$person = $connection->builder()->table('persons')->where('id', '=', 1)->first(['name', 'email']);

Fetching the value of a single column is done using the column method.

	$email = $connection->builder()->table('persons')->where('id', '=', 1)->column('email');

If you need to process large ammounts of data then the ```Query::batch``` method will help you limit the memory usage of your application. The default batch size is a 1000 records but you can override this using the optional second parameter. 

You can also set the offset starting point and offset end point using the optional third and fourth parameters respectively. This is useful if you have parallel workers processing data.

	$connection->builder()->table('persons')->ascending('id')->batch(function($batch)
	{
		// Process the batch here
	});

--------------------------------------------------------

<a id="inserting_data"></a>

### Inserting data

Inserting data is done using the ```Query::insert``` method.

	$connection->builder()->table('table')
	->insert(['field1' => 'foo', 'field2' => 'bar', 'field3' => new DateTime()]);

--------------------------------------------------------

<a id="updating_data"></a>

### Updating data

Updating data is done using the ```Query::update``` method.

	$connection->builder()->table('table')
	->where('id', '=', 10)
	->update(['field1' => 'foo', 'field2' => 'bar', 'field3' => time()]);

There are also shortcuts for incrementing and decrementing column values:

	$connection->builder()->table('article')->increment('views');

	$connection->builder()->table('article')->increment('views', 10);

	$connection->builder()->table('show')->decrement('tickets')

	$connection->builder()->table('show')->decrement('tickets', 50);

--------------------------------------------------------

<a id="deleting_data"></a>

### Deleting data

Deleting data is done using the ```Query::delete``` method.

	$connection->builder()->table('table')->where('id', '=', 10)->delete();

--------------------------------------------------------

<a id="aggregates"></a>

### Aggregates

The query builder also includes a few handy shortcuts to the most common aggregate functions:

	// Counting

	$count = $connection->builder()->table('persons')->count();

	$count = $connection->builder()->table('persons')->where('age', '>', 25)->count();

	// Average value

	$height = $connection->builder()->table('persons')->avg('height');

	$height = $connection->builder()->table('persons')->where('age', '>', 25)->avg('height');

	// Largest value

	$height = $connection->builder()->table('persons')->max('height');

	$height = $connection->builder()->table('persons')->where('age', '>', 25)->max('height');

	// Smallest value

	$height = $connection->builder()->table('persons')->min('height');

	$height = $connection->builder()->table('persons')->where('age', '>', 25)->min('height');

	// Sum

	$height = $connection->builder()->table('persons')->sum('height');

	$height = $connection->builder()->table('persons')->where('age', '>', 25)->sum('height');

--------------------------------------------------------

<a id="where_clauses"></a>

### WHERE clauses

where(), whereRaw(), orWhere(), orWhereRaw()

	// SELECT * FROM `persons` WHERE `age` > 25

	$persons = $connection->builder()->table('persons')
	->where('age', '>', 25)
	->all();

	// SELECT * FROM `persons` WHERE `age` > 25 OR `age` < 20

	$persons = $connection->builder()->table('persons')
	->where('age', '>', 25)
	->orWhere('age', '<', 20)
	->all();

	// SELECT * FROM `persons` WHERE (`age` > 25 AND `height` > 180) AND `email` IS NOT NULL

	$persons = $connection->builder()->table('persons')
	->where(function($query)
	{
		$query->where('age', '>', 25);
		$query->where('height', '>', 180);
	})
	->notNull('email')
	->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

<a id="where_between_clauses"></a>

### WHERE BETWEEN clauses

between(), orBetween(), notBetween(), orNotBetween()

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25

	$persons = $connection->builder()->table('persons')
	->between('age', 20, 25)
	->all();

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25 OR `age` BETWEEN 30 AND 35

	$persons = $connection->builder()->table('persons')
	->between('age', 20, 25)
	->orBetween('age', 30, 35)
	->all();

<a id="where_in_clauses"></a>

### WHERE IN clauses

in(), orIn(), notIn(), orNotIn()

	// SELECT * FROM `persons` WHERE `id` IN (1, 2, 3, 4, 5)

	$persons = $connection->builder()->table('persons')
	->in('id', [1, 2, 3, 4, 5])
	->all();

	// SELECT * FROM `persons` WHERE `id` IN (SELECT `id` FROM `persons` WHERE `id` != 1)

	$persons = $connection->builder()->table('persons')
	->in('id', function($query)
	{
		$query->table('persons')->where('id', '!=', 1)->columns(['id']);
	})
	->all();

<a id="where_is_null_clauses"></a>

### WHERE IS NULL clauses

null(), orNull(), notNull(), orNotNull()

	// SELECT * FROM `persons` WHERE `address` IS NULL

	$persons = $connection->builder()->table('persons')
	->null('address')
	->all();

<a id="where_exists_clauses"></a>

### WHERE EXISTS clauses

exists(), orExists(), notExists(), orNotExists()

	// SELECT * FROM `persons` WHERE EXISTS (SELECT * FROM `cars` WHERE `cars`.`person_id` = `persons`.`id`)

	$persons = $connection->builder()->table('persons')
	->exists(function($query)
	{
		$query->table('cars')->where('cars.person_id', '=', new Raw('persons.id');
	})
	->all();

<a id="join_clauses"></a>

### JOIN clauses

join(), joinRaw(), leftJoin(), leftJoinRaw()

	// SELECT * FROM `persons` INNER JOIN `phones` ON `persons`.`id` = `phones`.`user_id`

	$persons = $connection->builder()->table('persons')
	->join('phones', 'persons.id', '=', 'phones.user_id')
	->all();

	// SELECT * FROM `persons` AS `u` INNER JOIN `phones` AS `p` ON
	// `u`.`id` = `p`.`user_id` OR `u`.`phone_number` = `p`.`number`

	$persons = $connection->builder()->table('persons as u')
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

	$customers = $connection->builder()->table('orders')
	->groupBy('customer')
	->all(['customer', new Raw('SUM(price) as sum')]);

	// SELECT `customer`, `order_date`, SUM(`order_price`) as `sum` FROM `orders` GROUP BY `customer`, `order_date`

	$customers = $connection->builder()->table('orders')
	->groupBy(['customer', 'order_date'])
	->all(['customer', 'order_date', new Raw('SUM(price) as sum')]);

<a id="having_clauses"></a>

### HAVING clauses

having(), orHaving()

	// SELECT `customer`, SUM(`price`) AS `sum FROM `orders` GROUP BY `customer` HAVING SUM(`price`) < 2000

	$customers = $connection->builder()->table('orders')
	->groupBy('customer')
	->having(new Raw('SUM(price)'), '<', 2000)
	->all(['customer', new Raw('SUM(price) as sum')]);

<a id="order_by_clauses"></a>

### ORDER BY clauses

orderBy(), orderByRaw(), descending(), descendingRaw(), ascending(), ascendingRaw()

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $connection->builder()->table('persons')
	->orderBy('name', 'asc')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $connection->builder()->table('persons')
	->ascending('name')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` DESC

	$persons = $connection->builder()->table('persons')
	->descending('name')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC, `age` DESC

	$persons = $connection->builder()->table('persons')
	->orderBy('name', 'asc')
	->orderBy('age', 'desc')
	->all();

	// SELECT * FROM `persons` ORDER BY `name`, `age` ASC

	$persons = $connection->builder()->table('persons')
	->orderBy(['name', 'age'], 'asc')
	->all();

> Make sure not to create SQL injection vectors when using raw sql in your query builder queries!

<a id="limit_and_offset_clauses"></a>

### LIMIT and OFFSET clauses

limit(), offset(), paginate()

	// SELECT * FROM `persons` LIMIT 10

	$persons = $connection->builder()->table('persons')
	->limit(10)
	->all();

	// SELECT * FROM `persons` LIMIT 10 OFFSET 10

	$persons = $connection->builder()->table('persons')
	->limit(10)
	->offset(10)
	->all();

You can also use a [pagination instance](:base_url:/docs/:version:/learn-more:pagination) to limit your results.

	// SELECT * FROM `persons` LIMIT 10 OFFSET 0

	$persons = $connection->builder()->table('persons')->paginate($pagination)->all();