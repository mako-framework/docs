# Collections

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Collections are, as the name suggests, objects that contain a collection of values. You can iterate over collections and access items just like you would with an array. In addition to this the class also implements a set of useful methods.

> The result set class of the [query builder](:base_url:/docs/:version:/databases-sql:query-builder) extends the collection class.

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

Creating a collection is easy. You can create an empty collection or pass a set of values to the constructor.

```
$collection1 = new Collection;

$collection2 = new Collection([1, 2, 3]);
```

Adding a value to the collection is done using the `put` method.

```
$collection = new Collection;

$collection->put(0, 'foo');

$collection[1] = 'bar';
```

You can check if an item exists in the collection using the `has` method.

```
$collection = new Collection(['foo' => 'bar']);

var_dump($collection->has('foo')); // true

var_dump($collection->has('bar')); // false
```

> You can also use the collection as an array with the `isset` function but it will return `false` for `null` values while the `has` method will return `true` as long as the key exists.

The `get` method can be used to retrieve an item from the collection. You can also specify a default value to return if the item doesn't exist using the optional second parameter. It is also possible to retrieve items by using the collection as an array.

```
$collection = new Collection(['foo' => 'bar']);

var_dump($collection->get('foo')); // 'bar'

var_dump($collection->get('bar', 'baz')); // 'baz'

var_dump($collection['foo']); // 'bar'
```

> Accessing a non-existing key when using the collection as an array will result in a `OutOfBoundsException` exception.

The `first` method will return the first item in the collection or `null` if the collection is empty.

```
$collection = new Collection(['foo', 'bar']);

$first = $collection->first(); // foo
```

The `last` method will return the last item in the collection or `null` if the collection is empty.

```
$collection = new Collection(['foo', 'bar']);

$last = $collection->last(); // bar
```

Deleting an item from the collection can be done using the `remove` method or by using the `unset` function while using the collection as an array.

```
$collection = new Collection(['foo' => 'bar', 'baz' => 'bax']);

$collection->remove('foo');

unset($collection['baz']);
```

If you want to delete all items from the collection then you'll have to use the `clear` method.

```
$collection = new Collection(['foo' => 'bar', 'baz' => 'bax']);

$collection->clear();
```

Appending a value to the collection is done using the `push` method or by treating the collection as an array.

```
$collection = new Collection;

$collection->push('foo');

$collection[] = 'bar';
```

Both the `getItems` and `getValues` methods will return an array containing all of the collection items. The difference is that `getItems` will maintain the keys while `getValues` will not.

```
$collection = new Collection([1 => 'foo', 2 => 'bar']);

var_dump($collection->getItems()); // [1 => 'foo', 2 => 'bar']

var_dump($collection->getValues()); // [0 => 'foo', 1 => 'bar']
```

> You can also reset the keys in the collection using the `resetKeys` method.

Prepending values to the collection can be done using the `unshift` method.

```
$collection = new Collection([2]);

$collection->unshift(1);
```

Shifting a value off of the collection can be done with the `shift` method.

```
$collection = new Collection([1, 2, 3]);

var_dump($collection->shift()); // 1

var_dump($collection->shift()); // 2

var_dump($collection->shift()); // 3
```

Popping a value off of the collection is done using the `pop` method.

```
$collection = new Collection([1, 2, 3]);

var_dump($collection->pop()); // 3

var_dump($collection->pop()); // 2

var_dump($collection->pop()); // 1
```

You can get the number of items in the collection by using the `count` method or by using the `count` function on the collection.

```
$collection = new Collection(['foo' => 'bar', 'baz' => 'bax']);

var_dump($collection->count()); // 2

var_dump(count($collection)); // 2
```

Checking if a collection is empty is done using the `isEmpty` method.

```
$collection = new Collection([1, 2, 3]);

var_dump($collection->isEmpty()); // false

$collection->clear();

var_dump($collection->isEmpty()); // true
```

You can sort items in a collection using the `sort` method.

```
$collection = new Collection([2, 1, 3, 5, 6, 4]);

$collection->sort(function ($a, $b)
{
	if($a == $b)
	{
		return 0;
	}

	return ($a < $b) ? -1 : 1;
});
```

> Index association will be maintained by default but you can set the optional second parameter to `false` to disable this.

A collection can be chunked into a collection of `n` sized collections using the `chunk` method. This can be useful when rendering content in a grid system like the one available in [Bootstrap](https://getbootstrap.com).

```
$collection = new Collection([1, 2, 3, 4, 5, 6]);

$chunked = $collection->chunk(2);
```

You can shuffle the contents of a collection using the `shuffle` method.

```
$collection = new Collection([1, 2, 3, 4, 5, 6]);

$collection->shuffle();
```

The `each` method allows you to modify your collection items using a callable.

```
$collection = new Collection([1, 2, 3]);

$collection->each(fn ($value, $key) => $key . ':' . $value);
```

The `map` method works just like the `each` method except that it returns a new collection instead of modifying the existing one

```
$collection = new Collection([1, 2, 3]);

$newCollection = $collection->map(fn ($value, $key) => $key . ':' . $value);
```

You can filter a collection and remove unwanted items using a callable with the `filter` method. The callable can also accept a second parameter if you need to filter items using the key. If no callable is passed then it will just filter out all [`falsy`](https://php.net/manual/en/language.types.boolean.php#language.types.boolean.casting) values.

```
$collection = new Collection([1, 2, 3, 'foo', 4]);

$filtered = $collection->filter(fn ($value) => is_int($value));
```

You can create a new collection where items not included in the whitelist have been removed using the `whith` method.

```
$collection1 = new Collection(['foo' => 1, 'bar' => 2, 'baz' => 3]);

$collection2 = $collection1->with(['foo', 'baz']);
```

If you want to use a blacklist instead of a whitelist then you can use the `without` method.

```
$collection1 = new Collection(['foo' => 1, 'bar' => 2, 'baz' => 3]);

$collection2 = $collection1->without(['bar']);
```

Two collections can be merged using the `merge` method.

```
$collection1 = new Collection([1, 2, 3]);

$collection2 = new Collection([4, 5, 6]);

$merged = $collection1->merge($collection2);
```
