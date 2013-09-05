# Caching

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The cache class provides a simple and consistent interface to the most common caching solutions:

* APC
* Database
* File
* Memcache / Memcached
* Memory
* Redis
* XCache
* ZendDisk
* ZendMemory
* WinCache

> The memory cache adapter is non-persistent and will be cleared between each request.

--------------------------------------------------------

<a id="usage"></a>

### Usage

To get an instance of the default cache just call the ```instance``` method of the cache class. If you want to get an instance of any of the other cache configurations defined in the config file then you'll have to pass the configuration name as the first argument to the instance method.

	// Returns instance of the "default" cache configuration defined in the config file

	$cache = Cache::instance();

	// Returns instance of the "apc" cache configuration defined in the config file

	$cache = Cache::instance('apc');

#### Writing

To write something to the cache use the ```write``` method. The method returns TRUE on success or FALSE on failure.

	// Stores the array for one hour

	$cache->write('my_array', array(1, 2, 3, 4), 3600);

	// You can also write to the default instance like this

	Cache::write('my_array', array(1, 2, 3, 4), 3600);

	// Data can also be stored using overloading but you will not be able to set the ttl

	$cache->my_array = array(1, 2, 3, 4);

#### Reading

To retrieve data that you have stored in the cache use the ```read``` method. The method returns FALSE if the cached data has expired or if it is not found.

	$myArray = $cache->read('my_array');

	// You can also read from the default instance like this

	$myArray = Cache::read('my_array');

	// Data can also be retrieved using overloading

	$myArray = $cache->my_array;

You can check whether or not an item exists in the cache by using the ```has``` method.

	if($cache->has('foo'))
	{
		// Do something
	}

	// You can also check if an item exists in the default instance like this

	if(Cache::has('foo'))
	{
		// Do something
	}

The ```remember``` method works as a combination of the read and write methods. It will return the item from cache if it exists and store it before returning it if it doesn't exist. Using a closure (anonymous function) as the value ensures that any resource intensive code doesn't get executed unless necessary.

	$stars = $cache->remember('stars', function()
	{
		return Database::table('universe')->sum('stars');
	}, 3600);

	// You can also use the remember method against the default instance like this

	$stars = Cache::remember('stars', function()
	{
		return Database::table('universe')->sum('stars');
	}, 3600);

#### Deleting

Deleting data is done by using the ```delete``` method. The method returns TRUE on success or FALSE on failure.

	$cache->delete('my_array');

	// You can also delete from the default instance like this

	Cache::delete('my_array');

	// Data can also be deleted using overloading

	unset($cache->my_array);

You can clear the entire cache by using the ```clear``` method. The method returns TRUE on success or FALSE on failure. Note that clearing the cache might also affect other applications.

	$cache->clear();

	// You can also clear the default instance like this

	Cache::clear();

> Clearing the cache can affect other applications when using shared memory caching solutions such as APC and Memcache.

#### Incrementing & decrementing

You increment a value in the cache by using the ```increment``` method.

	$cache->increment('foo');

	// You can also increment a value in the default instance like this

	Cache::increment('foo');

> There are minor discrepancies between the different increment method implementations in regards to expiration times.

You decrement a value in the cache by using the ```decrement``` method.

	$cache->decrement('foo');

	// You can also decrement a value in the default instance like this

	Cache::decrement('foo');

> There are minor discrepancies between the different decrement method implementations in regards to expiration times.