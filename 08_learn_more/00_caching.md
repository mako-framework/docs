# Caching

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage:basics)
	- [Incrementing and decrementing](#usage:incrementing_and_decrementing)
	- [Magic shortcut](#usage:magic_shortcut)

--------------------------------------------------------

The cache library provides a simple and consistent interface to the most common caching solutions:

* APCU
* Database
* File
* Memcache / Memcached
* Memory
* NullStore
* Redis
* ZendDisk
* ZendMemory
* WinCache

> The memory cache adapter is non-persistent and will be cleared between each request. The null adapter will not store any data.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:basics"></a>

#### Basics

To get an instance of the default cache just call the `CacheManager::instance` method. If you want to get an instance of any of the other cache configurations defined in the config file then you'll have to pass it the configuration name.

```
// Returns instance of the "default" cache configuration defined in the config file

$cache = $this->cache->instance();

// Returns instance of the "apc" cache configuration defined in the config file

$cache = $this->cache->instance('apc');
```

Adding data to the cache is done using the `put` method. The method returns TRUE on success or FALSE on failure.

```
$cache->put('my_array', [1, 2, 3, 4], 3600);
```

You can also add data to the cache only if it doesn't already exist by using the `putIfNotExists` method.

```
$cache->putIfNotExists('my_array', [1, 2, 3, 4], 3600);
```

You can check whether or not an item exists in the cache by using the `has` method.

```
if($cache->has('foo'))
{
	// Do something
}
```

The `get` method is used to retrieve data from the cache. The method returns FALSE if the cached data has expired or if it is not found.

```
$cached = $cache->get('my_array');
```

The `getOrElse` method returns the data if the key exists and caches the return value of the closure if it doesn't. The closure does not get executed if the key exists in the cache.

```
$cached = $cache->getOrElse('foo', function()
{
	sleep(5);

	return 'this will get cached for 30 seconds';
}, 30);
```

The `getAndPut` method allows you to retrieve a cached value and replace it with a new value in a single call. The method returns FALSE if the cached data has expired or if it is not found.

```
$cached = $cache->getAndPut('my_array', [1, 3, 4, 5], 3600);
```

If you need to retrieve and remove a value then you can use the `getAndRemove` method. The method returns FALSE if the cached data has expired or if it is not found.

```
$cached = $cache->getAndRemove('my_array');
```

Removing data from the cache is done by using the `remove` method. The method returns TRUE on success or FALSE on failure.

```
$cache->remove('my_array');
```

You can clear the entire cache by using the `clear` method. The method returns TRUE on success or FALSE on failure. Note that clearing the cache might also affect other applications.

```
$cache->clear();
```

> Clearing the cache can affect other applications when using shared memory caching solutions such as APC and Memcache.
{.warning}

<a id="usage:incrementing_and_decrementing"></a>

#### Incrementing and decrementing

The `APCU`, `Memcached`, `Memory`, `NullStore` and `Redis` stores implement the `IncrementDecrementInterface` interface.

The `increment` method allows you to increment a stored number.

```
$incremented = $cache->increment('counter');
```

The `decrement` method allows you to decrement a stored number.

```
$incremented = $cache->increment('counter');
```

Both methods also have an optional second parameter that allows you to set amount by which to increment or decrement the value.

<a id="usage:magic_shortcut"></a>

#### Magic shortcut

You can access the default cache instance directly without having to go through the `instance` method thanks to the magic `__call` method.

```
$cached = $this->cache->get('my_array');
```
