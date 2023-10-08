# Query builder

--------------------------------------------------------

* [Getting a query builder instance](#getting_a_query_builder_instance)
* [Fetching data](#fetching_data)
* [Inserting data](#inserting_data)
* [Updating data](#updating_data)
* [Deleting data](#deleting_data)
* [Aggregates](#aggregates)
* [Clauses](#clauses)
	* [WHERE clauses](#clauses:where_clauses)
	* [WHERE BETWEEN clauses](#clauses:where_between_clauses)
	* [WHERE IN clauses](#clauses:where_in_clauses)
	* [WHERE IS NULL clauses](#clauses:where_is_null_clauses)
	* [WHERE EXISTS clauses](#clauses:where_exists_clauses)
	* [JOIN clauses](#clauses:join_clauses)
	* [GROUP BY clauses](#clauses:group_by_clauses)
	* [HAVING clauses](#clauses:having_clauses)
	* [ORDER BY clauses](#clauses:order_by_clauses)
	* [LIMIT and OFFSET clauses](#clauses:limit_and_offset_clauses)
* [Common table expressions](#common_table_expressions)
* [Set operations](#set_operations)
* [Row-level locking](#row_level_locking)
* [Dialect specific SQL](#dialect_specific_sql)
* [JSON data](#JSON_data)
* [Array and JSON representations of results](#array_and_json_representations)

--------------------------------------------------------

The query builder allows you to programmatically build SQL queries.

> All queries executed by the query builder use prepared statements, thus mitigating the risk of SQL injections. However, you have to make sure that you don't create SQL injection vectors if you're using using raw SQL in your query builder queries!

The query builder currently supports the following dialects:

* Firebird
* MariaDB
* MySQL
* Oracle
* PostgreSQL
* SQLite
* SQLServer

> The example SQL in the documentation is generated using the MySQL compiler.

--------------------------------------------------------

### <a id="getting_a_query_builder_instance" href="#getting_a_query_builder_instance">Getting a query builder instance</a>

You can create a query builder instance using the `Connection::getQuery()` method.

```
$query = $connection->getQuery();
```

--------------------------------------------------------

### <a id="fetching_data" href="#fetching_data">Fetching data</a>

If you only want to retrieve a single row then you can use the `first` method.

```
$person = $query->table('persons')->where('id', '=', 1)->first();
```

> Note that the `first` method will return `null` if nothing is found.

If you want to throw an exception if there isn't a matching record then you can use the `firstOrThrow` method.

```
// By default if throws a mako\database\exceptions\NotFoundException

$person = $query->table('persons')->where('id', '=', 1)->firstOrThrow();

// But you can make it throw any exception you want 
// You can for example throw a mako\http\exceptions\NotFoundException to display a 404 page

$person = $query->table('persons')->where('id', '=', 1)->firstOrThrow(NotFoundException::class);
```

Fetching all rows is done using the `all` method.

```
$persons = $query->table('persons')->all();
```

You can also specify which columns you want to include in the result set

```
$persons = $query->table('persons')->select(['name', 'email'])->all();
```

To make a distinct selection use the `distinct` method

```
$persons = $query->table('persons')->select(['name', 'email'])->distinct()->all();
```

> Note that the `all` and `paginate` methods return a result set object and not an array so you'll need to use the `isEmpty` method to check if it's empty.

Selecting from the results of a subquery is also possible.

```
$persons = $query->table(new Subquery(function ($query) {
	$query->table('persons')->select(['name'])->distinct();
}, 'distinct_names'))
->where('name', '!=', 'John Doe')
->all();
```

You can also use the `as` method of the `Subquery` to set the subquery table alias.

```
$persons = $query->table((new Subquery(function ($query) {
	$query->table('persons')->select(['name'])->distinct();
}))->as('distinct_names'))
->where('name', '!=', 'John Doe')
->all();
```

Advanced column selections can also be made using raw SQL and subqueries.

```
$persons = $query->table('persons')->select([
	'name',
	'email',
	new Raw("CASE gender WHEN 'm' THEN 'male' ELSE 'female' END AS gender"),
	new Subquery(function ($query) {
		$query->table('persons')->select([new Raw('AVG(age)']));
	}, 'average_age')
])
->all();
```

If you need to process a large dataset and don't want to put the entire result set in memory then you can use the `yield` method. It returns a generator that lets you iterate over the result set.

```
$persons = $query->table('persons')->select(['name', 'email'])->yield();

foreach ($persons as $person) {
	// Only a single row is kept in memory at a time
}
```

> Note that when using MySQL you might have to configure PDO to use [unbuffered queries](https://php.net/manual/en/mysqlinfo.concepts.buffering.php) for this to work as expected.
{.warning}

In addition to using the `yield` method to process large amounts of data you can also use the `batch` method. The default batch size is a 1000 records but you can override this using the optional second parameter.

You can also set the offset starting point and offset end point using the optional third and fourth parameters respectively. This is useful if you have parallel workers processing data.

```
$query->table('persons')->ascending('id')->batch(function ($batch) {
	// Process the batch here
});
```

Fetching the value of a single column is done using the `column` method.

```
$email = $query->table('persons')->select(['email'])->where('id', '=', 1)->column();

// You can also use the following syntax

$email = $query->table('persons')->where('id', '=', 1)->column('email');
```

It is also possible to fetch an array containing the values of a single column using the `columns` method.

```
 $emails = $query->table('persons')->select(['email'])->columns();

 // You can also use the following syntax

 $emails = $query->table('persons')->columns('email');
 ```

The `pairs` method allows you to fetch an array where the first column is used as the array keys and the second is used as the array values.

```
$pairs = $query->table->('users')->pairs('id', 'email');
```

--------------------------------------------------------

### <a id="inserting_data" href="#inserting_data">Inserting data</a>

Inserting data is done using the `insert` method.

```
$query->table('foobars')->insert(['field1' => 'foo', 'field2' => new DateTime]);
```

You can also insert data using the `insertAndGetId` method. It will create the record and return the generated auto increment id.

```
$query->table('foobars')->insertAndGetId(['field1' => 'foo', 'field2' => new DateTime]);
```

> When working with [PostgreSQL](https://www.postgresql.org) the `insertAndGetId` method assumes that the sequence follows the default naming convention (`<table_name>_<primary_key_name>_seq`) You can override the default primary key name (`id`) by using the optional second parameter.

--------------------------------------------------------

### <a id="updating_data" href="#updating_data">Updating data</a>

Updating data is done using the `update` method.

```
$query->table('foobars')
->where('id', '=', 10)
->update(['field1' => 'foo', 'field2' => new DateTime]);
```

There are also shortcuts for incrementing and decrementing column values:

```
$query->table('articles')->where('id', '=', 1)->increment('views');

$query->table('articles')->where('id', '=', 1)->increment('views', 10);

$query->table('shows')->where('id', '=', 1)->decrement('tickets')

$query->table('shows')->where('id', '=', 1)->decrement('tickets', 50);
```

--------------------------------------------------------

### <a id="deleting_data" href="#deleting_data">Deleting data</a>

Deleting data is done using the `delete` method.

```
$query->table('articles')->where('id', '=', 10)->delete();
```

--------------------------------------------------------

### <a id="aggregates" href="#aggregates">Aggregates</a>

The query builder also includes a few handy shortcuts to the most common aggregate functions:

```
// Counting

$count = $query->table('persons')->count();

$count = $query->table('persons')->where('age', '>', 25)->count();

// Distinct counting

$count = $query->table('persons')->countDistinct('age');

$count = $query->table('persons')->countDistinct(['age', 'height']);

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
```

--------------------------------------------------------

### <a id="clauses" href="#clauses">Clauses</a>

#### <a id="clauses:where_clauses" href="#clauses:where_clauses">WHERE clauses</a>

where(), orWhere(), whereColumn(), orWhereColumn(), whereRaw(), orWhereRaw(), whereDate(), orWhereDate()

```
// SELECT * FROM `persons` WHERE `age` > 25

$persons = $query->table('persons')->where('age', '>', 25)->all();

// SELECT * FROM `persons` WHERE `age` > 25 OR `age` < 20

$persons = $query->table('persons')->where('age', '>', 25)->orWhere('age', '<', 20)->all();

// SELECT * FROM `persons` WHERE (`username`, `email`) = ('foo', 'foobar@example.org') LIMIT 1

$person = $query->table('persons')->where(['username', 'email'], '=', ['foo', 'foo@example.org'])->first();

// SELECT * FROM `persons` WHERE (`age` > 25 AND `height` > 180)

$persons = $query->table('persons')
->where(function ($query) {
	$query->where('age', '>', 25);
	$query->where('height', '>', 180);
})
->all();
```

The `whereColumn` and `orWhereColumn` methods allow you to compare two columns.

```
// SELECT * FROM `persons` WHERE `first_name` = `last_name`

$persons = $query->table('persons')->whereColumn('first_name', '=', 'last_name')->all();
```

The `wereRaw` and `orWhereRaw` methods allow you to set a "raw" parameter value or to write an entire sql expression.

```
// SELECT * FROM `persons` WHERE `age` > AVG(`age`)

$persons = $query->table('persons')->whereRaw('age', '>', 'AVG(`age`)')->all();

// SELECT * FROM `persons` WHERE MATCH(`name`) AGAINST ('foobar' IN BOOLEAN MODE)

$persons = $query->table('persons')->whereRaw('MATCH(`name`) AGAINST (? IN BOOLEAN MODE)', ['foobar']);
```

The `whereDate` and `orWhereDate` methods allow you to easily match records based on the date portion of a datetime column. The methods accept dates in the `YYYY-MM-DD` format and instances of `DateTimeInterface`.

```
// SELECT * FROM `articles` WHERE `created_at` > '2019-07-08 23:59:59.999999'

$articles = $query->table('articles')->whereDate('created_at', '>', '2019-07-08')->all();
```

#### <a id="clauses:where_between_clauses" href="#clauses:where_between_clauses">WHERE BETWEEN clauses</a>

between(), orBetween(), notBetween(), orNotBetween(), betweenDate(), orBetweenDate(), notBetweenDate(), orNotBetweenDate()

```
// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25

$persons = $query->table('persons')->between('age', 20, 25)->all();

// SELECT * FROM `persons` WHERE `age` BETWEEN 20 AND 25 OR `age` BETWEEN 30 AND 35

$persons = $query->table('persons')->between('age', 20, 25)->orBetween('age', 30, 35)->all();
```

The `betweenDate`, `orBetweenDate`, `notBetweenDate` and `orNotBetweenDate` methods make it easy to match records between two dates using the date portion of a datetime column. The methods accept dates in the `YYYY-MM-DD` format and instances of `DateTimeInterface`.

```
// SELECT * FROM `articles` WHERE `created_at` 
// BETWEEN '2019-07-01 00:00:00.000000' AND '2019-07-31 23:59:59.999999'

$articles = $query->table('articles')->betweenDate('created_at', '2019-07-01', '2019-07-31')->all();
```

#### <a id="clauses:where_in_clauses" href="#clauses:where_in_clauses">WHERE IN clauses</a>

in(), orIn(), notIn(), orNotIn()

```
// SELECT * FROM `persons` WHERE `id` IN (1, 2, 3, 4, 5)

$persons = $query->table('persons')->in('id', [1, 2, 3, 4, 5])->all();

// SELECT * FROM `persons` WHERE `id` IN (SELECT `id` FROM `persons` WHERE `id` != 1)

$persons = $query->table('persons')
->in('id', new Subquery(function ($query) {
	$query->table('persons')->select(['id'])->where('id', '!=', 1);
}))
->all();
```

#### <a id="clauses:where_is_null_clauses" href="#clauses:where_is_null_clauses">WHERE IS NULL clauses</a>

isNull(), orIsNull(), isNotNull(), orIsNotNull()

```
// SELECT * FROM `persons` WHERE `address` IS NULL

$persons = $query->table('persons')->isNull('address')->all();
```

#### <a id="clauses:where_exists_clauses" href="#clauses:where_exists_clauses">WHERE EXISTS clauses</a>

exists(), orExists(), notExists(), orNotExists()

```
// SELECT * FROM `persons` WHERE EXISTS (SELECT * FROM `cars` WHERE `cars`.`person_id` = `persons`.`id`)

$persons = $query->table('persons')
->exists(new Subquery(function ($query) {
	$query->table('cars')->whereRaw('cars.person_id', '=', 'persons.id');
}))
->all();
```

#### <a id="clauses:join_clauses" href="#clauses:join_clauses">JOIN clauses</a>

join(), joinRaw(), leftJoin(), leftJoinRaw(), rightJoin(), rightJoinRaw(), crossJoin(), lateralJoin()

```
// SELECT * FROM `persons` INNER JOIN `phones` ON `persons`.`id` = `phones`.`user_id`

$persons = $query->table('persons')->join('phones', 'persons.id', '=', 'phones.user_id')->all();

// SELECT * FROM `persons` AS `u` INNER JOIN `phones` AS `p` ON
// (`u`.`id` = `p`.`user_id` OR `u`.`phone_number` = `p`.`number`)

$persons = $query->table('persons as u')
->join('phones as p', function ($join) {
	$join->on('u.id', '=', 'p.user_id');
	$join->orOn('u.phone_number', '=', 'p.number');
})
->all();

// SELECT * FROM `drinks` CROSS JOIN `meals`

$menu = $query->table('drinks')->crossJoin('meals')->all();

// SELECT `customers`.*, `recent_sales`.* FROM `customers` LEFT OUTER JOIN LATERAL (
//	SELECT * FROM `sales` WHERE `sales`.`customer_id` = `customers`.`id` 
//	ORDER BY `created_at` DESC LIMIT 3
// ) AS `recent_sales` ON TRUE

$customers = $query->table('customers')
->lateralJoin(new Subquery(function (Query $query): void {
	$query->table('sales')
	->whereRaw('sales.customer_id', '=', 'customers.id')
	->descending('created_at')
	->limit(3);
}, 'recent_sales'))
->select(['customers.*', 'recent_sales.*'])->all();
```

#### <a id="clauses:group_by_clauses" href="#clauses:group_by_clauses">GROUP BY clauses</a>

groupBy()

```
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
```

#### <a id="clauses:having_clauses" href="#clauses:having_clauses">HAVING clauses</a>

having(), havingRaw(), orHaving(), orHavingRaw()

```
// SELECT `customer`, SUM(`price`) AS `sum` FROM `orders` GROUP BY `customer` HAVING SUM(`price`) < 2000

$customers = $query->table('orders')
->select(['customer', new Raw('SUM(price) as sum')])
->groupBy('customer')
->havingRaw('SUM(price)', '<', 2000)
->all();
```

#### <a id="clauses:order_by_clauses" href="#clauses:order_by_clauses">ORDER BY clauses</a>

orderBy(), orderByRaw(), descending(), descendingRaw(), ascending(), ascendingRaw()

```
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
```

#### <a id="clauses:limit_and_offset_clauses" href="#clauses:limit_and_offset_clauses">LIMIT and OFFSET clauses</a>

limit(), offset(), paginate()

```
// SELECT * FROM `persons` LIMIT 10

$persons = $query->table('persons')->limit(10)->all();

// SELECT * FROM `persons` LIMIT 10 OFFSET 10

$persons = $query->table('persons')->limit(10)->offset(10)->all();
```

You can also use the [`paginate` method](:base_url:/docs/:version:/learn-more:pagination#usage_with_the_query_builder) to limit your results.

```
// SELECT * FROM `persons` LIMIT 10 OFFSET 0

$persons = $query->table('persons')->paginate(10);
```

--------------------------------------------------------

### <a id="common_table_expressions" href="#common_table_expressions">Common table expressions</a>

with(), withRecursive()

The `with` method allows you to add common table expressions to your queries.

```
// WITH `cte` AS (SELECT 1, 2, 3) SELECT * FROM `cte`

$result = $query->with('cte', [], new Subquery(function ($query) {
	$query->selectRaw('1, 2, 3');
}))->table('cte')->all();

// WITH `cte` (`one`, `two`, `three`) AS (SELECT 1, 2, 3) SELECT * FROM `cte`

$result = $query->with('cte', ['one', 'two', 'three'], new Subquery(function ($query) {
	$query->selectRaw('1, 2, 3');
}))
->table('cte')->all();
```

You can also add recursive common table expressions using the `withRecursive` method.

```
// WITH RECURSIVE `cte` AS (SELECT 1, 2, 3) SELECT * FROM `cte`

$result = $query->withRecursive('cte', [], new Subquery(function ($query) {
	$query->selectRaw('1, 2, 3');
}))->table('cte')->all();

// WITH RECURSIVE `cte` (`one`, `two`, `three`) AS (SELECT 1, 2, 3) SELECT * FROM `cte`

$result = $query->withRecursive('cte', ['one', 'two', 'three'], new Subquery(function ($query) {
	$query->selectRaw('1, 2, 3');
}))
->table('cte')->all();
```

> The query builder allows you add multiple common table expressions to your queries and it even supports nesting.

--------------------------------------------------------

### <a id="set_operations" href="#set_operations">Set operations</a>

union(), unionAll(), intersect(), intersectAll(), except(), exceptAll()

You can also combine the results of multiple queries into a single result set using set operations.

```
// SELECT * FROM `sales2015` UNION SELECT * FROM `sales2016`

$result = $query->table('sales2015')->union()->table('sales2016')->all();
```

--------------------------------------------------------

### <a id="row_level_locking" href="#row_level_locking">Row-level locking</a>

lock(), sharedLock()

The `lock()` method can be used to enable row-level locking during database transactions.

```
// SELECT * FROM `persons` WHERE `age` = 30 FOR UPDATE

$persons = $query->table('persons')->where('age', '=', 30)->lock()->all();
```

It will use an exclusive lock by default but you can enable shared locking by passing `false` to the `lock()` method or by using the `sharedLock()` method.

```
// SELECT * FROM `persons` WHERE `age` = 30 LOCK IN SHARE MODE

$persons = $query->table('persons')->where('age', '=', 30)->lock(false)->all();
```

It is also possible to provide a custom locking clause.

```
// SELECT * FROM `persons` WHERE `age` = 30 CUSTOM LOCK

$persons = $query->table('persons')->where('age', '=', 30)->lock('CUSTOM LOCK')->all();
```

Here's an overview of the locking clauses generated for the different RDBMSes that support row-level locking.

| RDBMS      | Exclusive lock          | Shared lock                 |
|------------|-------------------------|-----------------------------|
| Firebird   | FOR UPDATE WITH LOCK    | WITH LOCK                   |
| MySQL      | FOR UPDATE              | LOCK IN SHARE MODE          |
| Oracle     | FOR UPDATE              | FOR UPDATE                  |
| PostgreSQL | FOR UPDATE              | FOR SHARE                   |
| SQLServer  | WITH (UPDLOCK, ROWLOCK) | WITH (HOLDLOCK, ROWLOCK)    |

> Row-level locking will gracefully degrade for any RDBMS that doesn't support the feature.

--------------------------------------------------------

### <a id="dialect_specific_sql" href="#dialect_specific_sql">Dialect specific SQL</a>

Sometimes you'll find yourself in situations where you have to use dialect specific features in your queries while still supporting multiple databases. This is where the `forCompiler` method comes in handy. 

The first parameter is the compiler class name and the second one is a closure where you can build upon the query.

```
$events = $query->table('events')
->forCompiler(PostgreSQL::class, function ($query) {
	$query->whereRaw('EXTRACT(YEAR FROM "date") = ?', ['1337']);
})
->forCompiler(MySQL::class, function ($query) {
	$query->whereRaw('YEAR(`date`) = ?', ['1337']);
})
->all();
```

--------------------------------------------------------

### <a id="JSON_data" href="#JSON_data">JSON data</a>

The query builder features a unified syntax for querying JSON data and it currently supports `MySQL`, `Oracle`, `PostgreSQL`, `SQLServer` and `SQLite`.

```
$foos = $query->table('articles')
->select(['meta->foo as foo'])
->where('meta->bar', '=', json_encode(1))
->all();
```

You can also use the unified syntax to update JSON values. This feature currently supports `MySQL`, `PostgreSQL` (jsonb), `SQLServer` and `SQLite`.

```
$query->table('articles')->update(['meta->bar' => json_encode(0)]);
```

--------------------------------------------------------

### <a id="array_and_json_representations" href="#array_and_json_representations">Array and JSON representations of results</a>

You can convert both single result and result set objects to arrays and JSON using the `toArray` and `toJson` methods respectively. JSON encoding of your results can also be achieved by using the [`json_encode`](https://php.net/manual/en/function.json-encode.php) function or by casting the objects to strings.

> The `toJson` method accepts the same optional option flags as the [`json_encode`](https://php.net/manual/en/function.json-encode.php) function.

```
$json = (string) $query->table('articles')->select(['id', 'title', 'content'])->limit(10)->all();
```

The code above will result in the following JSON:

```
[
	{"id": 1, "title": "Article 1", "content": "Article 1 content"},
	{"id": 2, "title": "Article 2", "content": "Article 2 content"},
	...
	{"id": 9, "title": "Article 9", "content": "Article 9 content"},
	{"id": 10, "title": "Article 10", "content": "Article 10 content"}
]
```
{.language-json}

Data fetched using the `paginate` method will return a JSON object instead of an array. The records are available as `data` while pagination information is available as `pagination`:

```
{
	"data": [
		{"id": 1, "title": "Article 1", "content": "Article 1 content"},
		{"id": 2, "title": "Article 2", "content": "Article 2 content"},
		...
		{"id": 9, "title": "Article 9", "content": "Article 9 content"},
		{"id": 10, "title": "Article 10", "content": "Article 10 content"}
	],
	"pagination": {
		"current_page": 1,
		"number_of_pages": 1,
		"items": 10,
		"items_per_page": 50,
		"first": "https:\/\/example.org\/api\/articles?page=1",
		"last": "https:\/\/example.org\/api\/articles?page=1",
		"next": null,
		"previous": null
	}
}
```
{.language-json}
