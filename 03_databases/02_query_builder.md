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

Fetching all rows is done using the ```all``` method.

	$persons = $connection->table('persons')->all();

	// You can also specify which columns you want to include in the result set

	$persons = $connection->table('persons')->all(array('name', 'email'));

	// To make a distinct selection use the distinct method

	$persons = $connection->table('persons')->distinct()->all(array('name', 'email'));

	// Selecting from the results of a subquery is also possible

	$persons = $connection->table
	(
		Database::subquery($connection->table('persons')->distinct()->columns(array('name')), 'distinct_names')
	)
	->where('name', '!=', 'John Doe')
	->all();

	// You can also make advanced column selections with raw SQL and subqueries

	$persons = $connection->table('persons')->all(array
	(
		'name',
		'email',
		Database::raw("CASE gender WHEN 'm' THEN 'male' ELSE 'female' END AS gender"),
		Database::subquery($connection->table('persons')->columns(array(Database::raw('AVG(age)'))), 'average_age')
	));

If you only want to retrieve a single row you can use the ```first``` method.

	$person = $connection->table('persons')->where('id', '=', 1)->first();

	// You can also specify which columns you want to include in the result set

	$person = $connection->table('persons')->where('id', '=', 1)->first(array('name', 'email'));

Fetching the value of a single column is done using the column method.

	$email = $connection->table('persons')->where('id', '=', 1)->column('email');

--------------------------------------------------------

<a id="inserting_data"></a>

### Inserting data

Inserting data is done using the ```insert``` method.

	$connection->table('table')
	->insert(array('field1' => 'foo', 'field2' => 'bar', 'field3' => time()));

--------------------------------------------------------

<a id="updating_data"></a>

### Updating data

Updating data is done using the ```update``` method.

	$connection->table('table')
	->where('id', '=', 10)
	->update(array('field1' => 'foo', 'field2' => 'bar', 'field3' => time()));

There are also shortcuts for incrementing and decrementing column values:

	$connection->table('article')->increment('views');

	$connection->table('article')->increment('views', 10);

	$connection->table('show')->decrement('tickets')

	$connection->table('show')->decrement('tickets', 50);

--------------------------------------------------------

<a id="deleting_data"></a>

### Deleting data

Deleting data is done using the ```delete``` method.

	$connection->table('table')->where('id', '=', 10)->delete();

--------------------------------------------------------

<a id="aggregates"></a>

### Aggregates

The query builder also includes a few handy shortcuts to the most common aggregate functions:

	// Counting

	$count = $connection->table('persons')->count();

	$count = $connection->table('persons')->where('age', '>', 25)->count();

	// Average value

	$height = $connection->table('persons')->avg('height');

	$height = $connection->table('persons')->where('age', '>', 25)->avg('height');

	// Largest value

	$height = $connection->table('persons')->max('height');

	$height = $connection->table('persons')->where('age', '>', 25)->max('height');

	// Smallest value

	$height = $connection->table('persons')->min('height');

	$height = $connection->table('persons')->where('age', '>', 25)->min('height');

	// Sum

	$height = $connection->table('persons')->sum('height');

	$height = $connection->table('persons')->where('age', '>', 25)->sum('height');

--------------------------------------------------------

<a id="where_clauses"></a>

### WHERE clauses

where(), orWhere()

	// SELECT * FROM `persons` WHERE `age` > 25

	$persons = $connection->table('persons')
	->where('age', '>', 25)
	->all();

	// SELECT * FROM `persons` WHERE `age` > 25 OR `age` < 20

	$persons = $connection->table('persons')
	->where('age', '>', 25)
	->orWhere('age', '<', 20)
	->all();

	// SELECT * FROM `persons` WHERE (`age` > 25 AND `height` > 180) AND `email` IS NOT NULL

	$persons = $connection->table('persons')
	->where(function($query)
	{
		$query->where('age', '>', 25);
		$query->where('height', '>', 180);
	})
	->notNull('email')
	->all();

<a id="where_between_clauses"></a>

### WHERE BETWEEN clauses

between(), orBetween(), notBetween(), orNotBetween()

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25

	$persons = $connection->table('persons')
	->between('age', 20, 25)
	->all();

	// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25 OR `age` BETWEEN 30 AND 35

	$persons = $connection->table('persons')
	->between('age', 20, 25)
	->orBetween('age', 30, 35)
	->all();

<a id="where_in_clauses"></a>

### WHERE IN clauses

in(), orIn(), notIn(), orNotIn()

	// SELECT * FROM `persons` WHERE `id` IN (1, 2, 3, 4, 5)

	$persons = $connection->table('persons')
	->in('id', array(1, 2, 3, 4, 5))
	->all();

	// SELECT * FROM `persons` WHERE `id` IN (SELECT `id` FROM `persons` WHERE `id` != 1)

	$persons = $connection->table('persons')
	->in('id', Database::subquery($connection->table('persons')->where('id', '!=', 1)->columns(array('id'))))
	->all();

<a id="where_is_null_clauses"></a>

### WHERE IS NULL clauses

null(), orNull(), notNull(), orNotNull()

	// SELECT * FROM `persons` WHERE `address` IS NULL

	$persons = $connection->table('persons')
	->null('address')
	->all();

<a id="where_exists_clauses"></a>

### WHERE EXISTS clauses

exists(), orExists(), notExists(), orNotExists()

	// SELECT * FROM `persons` WHERE EXISTS (SELECT * FROM `cars` WHERE `cars`.`person_id` = `persons`.`id`)

	$persons = $connection->table('persons')
	->exists
	(
		Database::subquery($connection->table('cars')->where('cars.person_id', '=', Database::raw('persons.id')))
	)
	->all();

<a id="join_clauses"></a>

### JOIN clauses

join(), leftJoin()

	// SELECT * FROM `persons` INNER JOIN `phones` ON `persons`.`id` = `phones`.`user_id`

	$persons = $connection->table('persons')
	->join('phones', 'persons.id', '=', 'phones.user_id')
	->all();

	// SELECT * FROM `persons` AS `u` INNER JOIN `phones` AS `p` ON
	// `u`.`id` = `p`.`user_id` OR `u`.`phone_number` = `p`.`number`

	$persons = $connection->table('persons as u')
	->join('phones as p', function($join)
	{
		$join->on('u.id', '=', 'p.user_id');
		$join->orOn('u.phone_number', '=', 'p.number');
	})
	->all();

<a id="group_by_clauses"></a>

### GROUP BY clauses

groupBy()

	// SELECT `customer`, `order_date`, SUM(`order_price`) as `sum` FROM `orders` GROUP BY `customer`

	$customers = $connection->table('orders')
	->groupBy('customer')
	->all(array('customer', Database::raw('SUM(price) as sum')));

	// SELECT `customer`, `order_date`, SUM(`order_price`) as `sum` FROM `orders` GROUP BY `customer`, `order_date`

	$customers = $connection->table('orders')
	->groupBy(array('customer', 'order_date'))
	->all(array('customer', 'order_date', Database::raw('SUM(price) as sum')));

<a id="having_clauses"></a>

### HAVING clauses

having(), orHaving()

	// SELECT `customer`, SUM(`price`) AS `sum FROM `orders` GROUP BY `customer` HAVING SUM(`price`) < 2000

	$customers = $connection->table('orders')
	->groupBy('customer')
	->having(Database::raw('SUM(price)'), '<', 2000)
	->all(array('customer', Database::raw('SUM(price) as sum')));

<a id="order_by_clauses"></a>

### ORDER BY clauses

orderBy(), descending(), ascending()

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $connection->table('persons')
	->orderBy('name', 'asc')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC

	$persons = $connection->table('persons')
	->ascending('name')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` DESC

	$persons = $connection->table('persons')
	->descending('name')
	->all();

	// SELECT * FROM `persons` ORDER BY `name` ASC, `age` DESC

	$persons = $connection->table('persons')
	->orderBy('name', 'asc')
	->orderBy('age', 'desc')
	->all();

	// SELECT * FROM `persons` ORDER BY `name`, `age` ASC

	$persons = $connection->table('persons')
	->orderBy(array('name', 'age'), 'asc')
	->all();

<a id="limit_and_offset_clauses"></a>

### LIMIT and OFFSET clauses

limit(), offset()

	// SELECT * FROM `persons` LIMIT 10

	$persons = $connection->table('persons')
	->limit(10)
	->all();

	// SELECT * FROM `persons` LIMIT 10 OFFSET 10

	$persons = $connection->table('persons')
	->limit(10)
	->offset(10)
	->all();