# Array helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The array helper contains methods that can be useful when working with arrays.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The `get` method returns a value from an array using "dot notation".

```
$array = ['foo' => ['bar' => 'baz']];

$bar = Arr::get($array, 'foo.bar');

// You can also specify a default value if the key doesn't exist

$baz = Arr::get($array, 'foo.baz', 'nope');
```

The `set` method sets an array value using "dot notation".

```
Arr::set($array, 'foo.baz', 'hello world');
```

The `delete` method deletes an array value using "dot notation".

```
Arr::delete($array, 'foo.bar');
```

The `random` method returns a random array value.

```
Arr::random(['green', 'blue', 'red', 'orange']);
```

The `isAssoc` method returns `true` if the array is associative and `false` if not.

```
// $assoc will be set to "false"

$assoc = Arr::isAssoc([1, 2, 3]);

// $assoc will be set to "true"

$assoc = Arr::isAssoc(['one' => 1, 'two' => 2, 'three' => 3]);
```

The `pluck` method returns the values from a single column of the input array, identified by the key.

```
$fruits =
[
	['name' => 'apple', 'color' => 'green'],
	['name' => 'banana', 'color' => 'yellow'];
];

$colors = Arr::pluck($fruits, 'color');
```
