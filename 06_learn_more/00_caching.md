# Caching

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The cache library provides a simple and consistent interface to the most common caching solutions:

* APC
* Database
* File
* Memcache / Memcached
* Memory
* Null
* Redis
* XCache
* ZendDisk
* ZendMemory
* WinCache

> The memory cache adapter is non-persistent and will be cleared between each request. The null adapter will not store any data.

--------------------------------------------------------

<a id="usage"></a>

### Usage

To get an instance of the default cache just call the ```CacheManager::instance``` method. If you want to get an instance of any of the other cache configurations defined in the config file then you'll have to pass it the configuration name.

	// Returns instance of the "default" cache configuration defined in the config file

	$cache = $this->cache->instance();

	// Returns instance of the "apc" cache configuration defined in the config file

	$cache = $this->cache->instance('apc');

Adding data to the cache is done using the ```put``` method. The method returns TRUE on success or FALSE on failure.

	// Stores the array for one hour

	$cache->put('my_array', [1, 2, 3, 4], 3600);

	// You can also add data to the default instance like this

	$this->cache->put('my_array', [1, 2, 3, 4], 3600);

The ```get``` method is used to retrieve data from the cache. The method returns FALSE if the cached data has expired or if it is not found.

	$myArray = $cache->get('my_array');

	// You can also read from the default instance like this

	$myArray = $this->cache->get('my_array');

You can check whether or not an item exists in the cache by using the ```has``` method.

	if($cache->has('foo'))
	{
		// Do something
	}

	// You can also check if an item exists in the default instance like this

	if($this->cache->has('foo'))
	{
		// Do something
	}

Removing data from the cache is done by using the ```remove``` method. The method returns TRUE on success or FALSE on failure.

	$cache->remove('my_array');

	// You can also remove data from the default instance like this

	$this->cache->remove('my_array');

You can clear the entire cache by using the ```clear``` method. The method returns TRUE on success or FALSE on failure. Note that clearing the cache might also affect other applications.

	$cache->clear();

	// You can also clear the default instance like this

	$this->cache->clear();

> Clearing the cache can affect other applications when using shared memory caching solutions such as APC and Memcache.