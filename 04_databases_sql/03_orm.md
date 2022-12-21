# ORM

--------------------------------------------------------

* [Naming conventions](#naming_conventions)
* [Basic usage](#basic_usage)
	- [CRUD](#basic_usage:crud)
	- [Selecting columns](#basic_usage:selecting_columns)
	- [Joins](#basic_usage:joins)
* [Relations](#relations)
	- [Has one](#relations:has_one)
	- [Has many](#relations:has_many)
	- [Belongs to](#relations:belongs_to)
	- [Many to many](#relations:many_to_many)
	- [Relation criteria](#relations:relation_criteria)
	- [Creating related records](#relations:creating_related_records)
	- [Eager loading](#relations:eager_loading)
	- [Counting related records](#relations:counting_related_records)
	- [Overriding naming conventions](#relations:overriding_naming_conventions)
* [Automatic typecasting](#automatic_typecasting)
	- [Scalars](#automatic_typecasting:scalars)
	- [DateTime](#automatic_typecasting:datetime)
	- [Enums](#automatic_typecasting:enums)
* [Mutators and accessors](#mutators_and_accessors)
* [Scopes](#scopes)
* [Mass assignment](#mass_assignment)
* [Cloning records](#cloning_records)
* [Array and JSON representations of results](#array_and_json_representations)
* [Traits](#traits)
	- [Camel cased data export and interaction](#traits:camel_cased_data_export_and_interaction)
	- [Nullable](#traits:nullable)
	- [Optimistic locking](#traits:optimistic_locking)
	- [Read-only records](#traits:read_only_records)
	- [Timestamped](#traits:timestamped)


--------------------------------------------------------

The ORM lets you map your database tables to objects and create relations between them.

--------------------------------------------------------

<a id="naming_conventions"></a>

### Naming conventions

The Mako ORM does not impose a naming standard on model class names but best practice is to use the camel cased singular form of the table name.

| Table name       | Model name       |
|------------------|------------------|
| articles         | Article          |
| import_jobs      | ImportJob        |

If you want to use the ORM on an existing database and don't want to rename all your tables then you can use the `$tableName` property to define the name of your table.

All tables are also expected to have an auto incrementing primary key column named `id`. The name of the primary key column can be configured using the `$primaryKey` property.

The ORM expects foreign key names to use the following pattern `<model name>_id` (e.g., item_id, user_id). This can be configured when setting up relations and we'll get back to this later on.

--------------------------------------------------------

<a id="key_types"></a>

### Key types

As previously mentioned, the ORM assumes that all your tables have an auto incrementing primary key by default. You can make it generate UUIDs, make your own custom key generator or tell it that your table doesn't have a primary key.

Use the `$primaryKeyType` property to define the key type.

| Key type          | Constant                           |
|-------------------|------------------------------------|
| Auto incrementing | ORM::PRIMARY_KEY_TYPE_INCREMENTING |
| UUID              | ORM::PRIMARY_KEY_TYPE_UUID         |
| Custom            | ORM::PRIMARY_KEY_TYPE_CUSTOM       |
| None              | ORM::PRIMARY_KEY_TYPE_NONE         |

> If you choose to use your own custom key generator then you'll have to implement the `generatePrimaryKey` method in your model class. You must also make sure that the generated value is unique.

--------------------------------------------------------

<a id="basic_usage"></a>

### Basic usage

<a id="basic_usage:crud"></a>

#### CRUD

Lets say you have a table called `articles` with three columns (id, title and content). These few lines of code is all you need to interact with the table:

```
<?php

namespace app\models;

use mako\database\midgard\ORM;

class Article extends ORM
{
	protected $tableName = 'articles';
}
```

Creating a new record is as simple as this:

```
$article = new Article;

$article->title   = 'Super awesome stuff';
$article->content = 'This is an article about some super awesome stuff.';

$article->save();
```

You can then fetch the article by its primary key value like this:

```
$article = Article::get(1);
```

> Note that the `get` method will return `null` if nothing is found.

If you want to throw an exception if there isn't a matching article then you can use the `getOrThrow` method.

```
// By default if throws a mako\database\exceptions\NotFoundException

$article = Article::getOrThrow(1);

// But you can make it throw any exception you want 
// You can for example throw a mako\http\exceptions\NotFoundException to display a 404 page

$article = Article::getOrThrow(1, exception: NotFoundException::class);
```

> We're using the PHP 8.0+ [named argument](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) syntax to skip the optional `$columns` parameter of the `getOrThrow` method.

The ORM is built on top of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder) so you can also use other criteria to find your record:

```
$article = (new Article)->where('title', '=', 'Super awesome stuff')->first();
```

Modifying an existing record is done like this:

```
$article = Article::get(1);

$article->title = 'New title';

$article->save();
```

And deleting a record is done like this:

```
$article = Article::get(1);

$article->delete();
```

<a id="basic_usage:selecting_columns"></a>

#### Selecting columns

By default the ORM selects all columns from the result set. You can specify the columns you want to select like this:

```
$articles = (new Article)->select(['id', 'title'])->all();
```

<a id="basic_usage:joins"></a>

#### Joins

You can also use joins when working with the ORM. In the following example we'll select all articles that have at least one comment:

```
$articles = (new Article)->join('comments', 'article.id', '=', 'comments.article_id')->all();
```

The code above will execute the following SQL:

```
SELECT `articles`.* FROM `articles` INNER JOIN `comments` ON `article`.`id` = `comments`.`article_id`
```

It will return duplicates for articles that have more than one comment. This can be solved using a distinct select:

```
$articles = (new Article)->distinct()->join('comments', 'article.id', '=', 'comments.article_id')->all();
```

--------------------------------------------------------

<a id="relations"></a>

### Relations

Being able to set up relations between tables is important when working with databases. The ORM supports `has one`, `belongs` to, `has many` and `many to many` relations.

<a id="relations:has_one"></a>

#### Has one

Lets create a user model and a profile model and set up a `has one` relation between them.

```
<?php

namespace app\models;

use mako\database\midgard\ORM;

class User extends ORM
{
	protected $tableName = 'users';

	public function profile()
	{
		return $this->hasOne(Profile::class);
	}
}
```

Lets not bother creating a relation in the profile model jus yet.

```
<?php

namespace app\models;

use mako\database\midgard\ORM;

class Profile extends ORM
{
	protected $tableName = 'profiles';
}
```

You can now access a users profile like this:

```
$user = User::get(1);

$profile = $user->profile;
```

<a id="relations:has_many"></a>

#### Has many

We can now add a `has many` relation to our user model.

```
public function articles()
{
	return $this->hasMany(Article::class);
}
```

We can now fetch all the articles that belong to the user like this:

```
$user = User::get(1);

$articles = $user->articles;
```

<a id="relations:belongs_to"></a>

#### Belongs to

The `belongs` to relation is the opposite of a `has one` or `has many` relation.

We can continue to build on the article model and add a `belongs` to relation. All we need to get this to work is add a foreign key column named `user_id` to the articles table.

```
public function user()
{
	return $this->belongsTo(User::class);
}
```

Fetching the user that owns the article can now be done line this:

```
$article = Article::get(1);

$user = $article->user;
```

<a id="relations:many_to_many"></a>

#### Many to many

The `many to many` relation requires a [junction table](https://en.wikipedia.org/wiki/Junction_table) between the two related tables. The name of the junction table should be the names of the two tables you want to join in alphabetical order separated by an underscore.

![junction table](:base_url:/assets/img/junctionTable.png)

The relation would then look like this in the user model:

```
public function groups()
{
	return $this->manyToMany(Group::class);
}
```

And like this in the group model:

```
public function users()
{
	return $this->manyToMany(User::class);
}
```

This is how you would use the relations:

```
// Fetch all the groups that the user belongs to

$user = User::get(1);

$groups = $user->groups;

// Fetch all the users that are in the group

$group = Group::get(1);

$users = $group->users;
```

<a id="relations:relation_criteria"></a>

#### Relation criteria

The ORM is built on top of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder) so you can add query criteria to your relations.

```
public function articles()
{
	return $this->hasMany(Article::class)->orderBy('title', 'asc');
}
```

They can be in the relation definition itself or you can add them when you're accessing the related records.

```
$articles = $user->articles()->orderBy('title', 'asc')->all();
```

<a id="relations:creating_related_records"></a>

#### Creating related records

The ORM lets you create related records without having to worry about remembering to set the right foreign key value.

```
$user = User::get(1);

$article = new Article();

$article->title   = 'My article title';
$article->content = 'My article content';

$user->articles()->create($article);
```

The article will now be saved and the value of the `user_id` foreign key will automatically be set to the users id. This method works for both `has one` and `has many` relations.

The `many to many` relation is a bit different since it requires a junction table. You'll have to use the `link` method to create a link between two related records.

```
$user = User::get(1);

$group = Group::get(1);

$user->groups()->link($group);

// This will produce the same result as the example above:

$user = User::get(1);

$group = Group::get(1);

$group->users()->link($user);
```

> You can also pass the primary key value of the record you want to link instead of the object.

Sometimes you'll need to store additional information in your junction table. This can easily be achieved by using the second parameter of the `link` method.

```
// Create a single link with attributes

$user->groups()->link(1, ['foo' => 'data']);

// Create two links with the same attributes

$user->groups()->link([1, 2], ['foo' => 'data']);

// Create two links with different attributes

$user->groups()->link([1, 2], [['foo' => 'data1'], ['foo' => 'data2']]);
```

You can also update the junction attributes by using the `updateLink` method.

```
// Update a single link

$user->groups()->updateLink(1, ['foo' => 'data']);

// Update two links with the same attributes

$user->groups()->updateLink([1, 2], ['foo' => 'data']);

// Update two links with different attributes

$user->groups()->updateLink([1, 2], [['foo' => 'data1'], ['foo' => 'data2']]);
```

Fetching the junction attributes is done using the `alongWith` method.

```
$groups = $user->groups()->alongWith(['foo'])->all();
```

The `unlink` method is used to remove the link between the records:

```
$user->groups()->unlink($group);

// This will produce the same result as the example above:

$group->users()->unlink($user);
```

<a id="relations:eager_loading"></a>

#### Eager loading

Loading related records can sometimes cause the `1 + N` query problem. This is where eager loading becomes handy.

```
foreach((new Comment)->limit(10)->all() as $comment)
{
	$comment->user->username;
}
```

The code above will execute `1` query to fetch `10` comments and then `1` query per iteration to retrieve the user who wrote the comment. This means that it has to execute `11` queries in total. Using eager loading can solve this problem.

```
foreach((new Comment)->including('users')->limit(10)->all() as $comment)
{
	$comment->user->username;
}
```

The code above will produce the same result as the previous example but it will only execute `2` queries instead of `11`.

You can eager load more relations using an array and nested relations using the dot syntax.

```
$articles = (new Article)->including(['user', 'comments', 'comments.user'])->limit(10)->all();
```

If you need to add query criteria to your relations then you can do so using a closure.

```
$articles = (new Article)->including(['user', 'comments as approved_comments' => function ($query)
{
	$query->where('approved', '=', true);
}, 'comments.user'])->limit(10)->all();
```

You can also define relations to eager load in the model definition using the `$including` property. This is useful if you know that you're going to need to eager load the relations more often than not.

```
protected $including = ['user', 'comments', 'comments.user'];
```

You can then disable eager loading of the relations if needed by using the `excluding` method:

```
$articles = (new Article)->excluding(['user', 'comments'])->limit(10)->all();
```

It is also possible to eager load relations using the `include` method on both model and result set instances. You can check if a model already has loaded a relation using the `includes` method.

```
$article = Article::get(1);

if(!$article->includes('comments'))
{
	$article->include(['comments', 'comments.user']);
}
```

<a id="relations:counting_related_records"></a>

#### Counting related records

Sometimes you'll want to count the number of related records without having to execute a second query. This can easily be achieved using the `withCountOf` method.

```
$articles = (new Article)->withCountOf('comments')->limit(10)->all();
```

Each `Article` object in the `$articles` result set will now have a `comments_count` property containing the number of comments related to each article.

> You can also pass an array of relation names if you want to count the number of related records for multiple relations.

If you want to add custom query criteria when counting related records then you can do so using a closure.

```
$articles = (new Article)->withCountOf(['comments AS approved_comments_count' => function ($query)
{
	$query->where('approved', '=', true);
}])->limit(10)->all();
```

<a id="relations:overriding_naming_conventions"></a>

#### Overriding naming conventions

The ORM relations rely on strict naming conventions but they can be overridden using the optional parameters of the relation methods. The first optional parameter lets you set the name of the foreign key. The `many to many` relation method has two additional parameters that let you set the junction table name and the junction key.

In the example below we are telling the relation to use a foreign key named `user` instead of the default, which should have been `user_id`.

```
public function articles()
{
	return $this->hasMany(Article::class, 'user');
}
```

--------------------------------------------------------

<a id="automatic_typecasting"></a>

### Automatic typecasting

You can configure your model to automatically typecast values on the way in and out of your database. This is done using the `$cast` property where the array key is the column name and the array value is the type you want to cast the column value to.

<a id="automatic_typecasting:scalars"></a>

#### Scalars

```
protected $cast = ['id' => 'int', 'published' => 'bool'];
```

Valid scalar types are `bool`, `int`, `float` and `string`.

> Note that the maximum value for `int` is `PHP_INT_MAX`.

<a id="automatic_typecasting:datetime"></a>

#### DateTime

The ORM and query builder both allow you to save dates as DateTime objects without first having to convert them to the appropriate format. Wouldn't it be nice if you could also automatically retrieve them as DateTime objects when fetching them from the database as well? This is possible thanks to the `date` typecast.

```
protected $cast = ['joined_at' => 'date', 'last_seen' => 'date'];
```

You'll now be able to treat the `joined_at` and `last_seen` values as [DateTime](:base_url:/docs/:version:/learn-more:date-and-time) objects.

```
$user = User::get(1);

$lastSeen = 'The user was last seen on ' . $user->last_seen->format('Y-m-d at H:i');
```

<a id="automatic_typecasting:enums"></a>

#### Enums

Both the query builder and ORM support enums values. You can automatically cast database values to the appropriate enum using the `enum` typecast.

```
protected $cast = ['transfer_status' => ['enum' => TransferStatus::class]];
```

> Note that the ORM can only cast to so-called [backed enums](https://www.php.net/manual/en/language.enumerations.backed.php) unless you implement your own `from` method.

--------------------------------------------------------

<a id="mutators_and_accessors"></a>

### Mutators and accessors

Mutators and accessors allow you to modify data on the way in and out of the database. Mutators are suffixed with `Mutator` and accessors and suffixed with `Accessor`.

The following mutator will encode the value when its assigned.

```
protected function numbersMutator(array $numbers)
{
	return json_encode($numbers);
}
```

You can assign the value like any normal value and it will be JSON-encoded internally in the model making it possible to store it in the database.

```
$model->numbers = [1, 2, 3, 4];
```

And the following accessor will decode the value when accessing it.

```
protected function numbersAccessor($numbers)
{
	return json_decode($numbers)
}
```

You can now retrieve the value like any normal value and it will automatically be JSON-decoded for you.

```
$arrayOfNumbers = $model->numbers;
```

--------------------------------------------------------

<a id="scopes"></a>

### Scopes

Scopes allow you to specify commonly used query criteria as methods. All scope methods must be suffixed with the `Scope` suffix.

```
public function publishedScope($query)
{
	$query->where('published', '=', 1);
}

public function popularAndPublishedScope($query, $minViewCount = 1000)
{
	$query->where('published', '=', 1)->where('views', '>', $minViewCount);
}
```

You can now retrieve published articles like this:

```
$articles = (new Article)->scope('published')->all();

$articles = (new Article)->scope('popularAndPublished')->all();

// Camel cased scope names can also be written using snake case

$articles = User::get(1)->articles()->scope('popular_and_published')->all();
```

Scopes also work through relations, and you can of course pass parameters to your scopes:

```
$articles = User::get(1)->articles()->scope('published')->all();

$articles = User::get(1)->articles()->scope('popularAndPublished', 2000)->all();
```

--------------------------------------------------------

<a id="mass_assignment"></a>

### Mass assignment

The ORM allows you to use mass assignment when creating or updating records. This can save you a few lines of code since you don't have to set each value individually but it can open attack vectors in your application if you're not careful.

```
// Create a new record using mass assignment

User::create($this->request->getPost()->all());

// Update an existing record using mass assignment

$user = User::get(1);

$user->assign($this->request->getPost()->all());

$user->save();
```

> The code above might seem like a good idea until a hacker adds an `is_admin` field to the POST data and gives himself admin privileges.
{.danger}

> You can make mass assignment a bit more secure by using the `$assignable` property and define a whitelist of fields that can be set through mass assignment. It's better to be safe than sorry so you should really only use this feature with trusted data.

--------------------------------------------------------

<a id="cloning_records"></a>

### Cloning records

You can clone records using the `clone` keyword:

```
$clone = clone User::get(1);

$clone->save();
```

You can also clone an entire result set:

```
$clones = clone (new User)->all();

foreach($clones as $clone)
{
	$clone->save();
}
```

--------------------------------------------------------

<a id="array_and_json_representations"></a>

### Array and JSON representations of results

You can convert both your result and result set objects to arrays and JSON when using the ORM [just like you can with plain query builder result and result set objects](:base_url:/docs/:version:/databases-sql:query-builder#array_and_json_representations).

> It is possible to exclude loaded columns and relations from the array and JSON representations by using the `$protected` property. You can also alter protection at runtime using the `protect()` and `expose()` methods.

--------------------------------------------------------

<a id="traits"></a>

### Traits

<a id="traits:camel_cased_data_export_and_interaction">

#### Camel cased data export and interation

Databases are usually designed with snake cased column and table names so interacting with columns and relations mapped to properties on ORM objects will lead to inconsistency in your code base if you use camel case everywhere else. This can be solved by using the `CamelCasedTrait` trait.

```
<?php

use mako\database\midgard\ORM;
use mako\database\midgard\traits\CamelCasedTrait;

class Upload extends ORM
{
	use CamelCasedTrait;
}

$upload = Upload::get(1);

// The database column is named "transfer_status" but can 
// now be accessed on the ORM object using "transferStatus"

$upload->transferStatus;
```

All column and relation names will be transformed to camel case when transforming ORM objects to arrays or JSON. 

If you only want to access data using camel cased properties but want to keep your array and json representations snake cased then you can use the `CamelCasedDataInteractionTrait` trait instead. If you prefer it the other way around then you can use the `CamelCasedDataExportTrait` trait.

> Note that all relations as well as accessors and mutators are expected to be defined using camel case when using the `CamelCasedTrait` and `CamelCasedDataInteractionTrait` traits.

<a id="traits:nullable"></a>

#### Nullable

If you have database columns that allow `null` values then you can use the `NullableTrait` to automatically replace empty strings with `null` when inserting or updating records.

All you have to do is to use the trait and configure your nullable columns using the `$nullable` property.

```
<?php

use mako\database\midgard\ORM;
use mako\database\midgard\traits\NullableTrait;

class Article extends ORM
{
	use NullableTrait;

	protected $nullable = ['source'];
}
```

<a id="traits:optimistic_locking"></a>

#### Optimistic locking

When two users are attempting to update the same record simultaneously, one of the updates will likely be overwritten. Optimistic locking can solve this problem.

To enable optimistic locking you need to use `OptimisticLockingTrait` trait. The database table also needs an integer column named `lock_version`. The name of the column can be configured using the `$lockingColumn` property.

```
<?php

use mako\database\midgard\ORM;
use mako\database\midgard\traits\OptimisticLockingTrait;

class Article extends ORM
{
	use OptimisticLockingTrait;
}
```

The second save in the example below will throw a `StaleRecordException` since the record is now outdated compared to the one stored in the database.

```
$article1 = Article::get(1);
$article2 = Article::get(1);

$article1->title = 'Foo';

$article1->save();

$article2->title = 'Bar';

$article2->save();
```

 The `reload` method can be used to refresh the outdated record.

```
$article2->reload();
```

The optimistic locking trait will also check for stale records when deleting.

> Note that optimistic locking only works when working with a single record.

<a id="traits:read_only_records"></a>

### Read-only records

You can make your records read-only by using the `ReadOnlyTrait`.

```
<?php

use mako\database\midgard\ORM;
use mako\database\midgard\traits\ReadOnlyTrait;

class User extends ORM
{
	use ReadOnlyTrait;
}
```

This will make it impossible to update or delete the records and a `ReadOnlyException` will be thrown if attempted.

```
// Load a read-only record

$user = User::get(1);

// Will throw a mako\database\midgard\traits\exceptions\ReadOnlyException

$user->delete();
```

<a id="traits:timestamped"></a>

#### Timestamped

You'll often want to track when a record has been created and when it was updated. The `TimestampedTrait` will do this for you automatically.

The trait requires you to add two `DATETIME` columns to your database table, `created_at` and `updated_at`. You can override the column names using the `$createdAtColumn` and `$updatedAtColumn` properties.

```
<?php

use mako\database\midgard\ORM;
use mako\database\midgard\traits\TimestampedTrait;

class Article extends ORM
{
	use TimestampedTrait;
}
```

You can touch the `updated_at` timestamp without having to modify any other data by using the `touch` method.

```
$article = Article::get(1);

$article->touch();
```

You can also make the ORM touch related records upon saving by listing the relations you want to touch in the `$touch` property.

```
protected $touch = ['foo', 'foo.bar']; // Nested relations are also supported
```

You can easily decide which type of changes that should touch related records using the `$shouldTouchOnInsert`, `$shouldTouchOnUpdate` and `$shouldTouchOnDelete` properties. All of them are set to `true` by default.
