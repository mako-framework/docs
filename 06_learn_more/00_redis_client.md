# Redis client

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The Redis class provides a simple and consistent way of communicating with a [Redis](http://redis.io) server.

--------------------------------------------------------

<a id="usage"></a>

### Usage

You create a Redis object by using the constructor. Use the optional name parameter if you don't want to create an object using the default Redis configuration.

	// Create an object using the default configuration

	$redis = new Redis();

	// Create an object using the 'my_other_server' configuration

	$redis = new Redis('my_other_server');

The ```pipeline``` method allows you to send multiple commands to the Redis server without having to wait for the replies. Using pipelining can be useful if you need to send a large number of commands as you will not be paying the cost of RTT for every call.

	$redis->set('x', 0);

	$replies = $redis->pipeline(function($redis)
	{
		for($i = 0; $i < 100; $i++)
		{
			$redis->incr('x');
		}
	});

The Redis class uses the magic __call method so every valid [Redis command](http://redis.io/commands) is a valid method.

	$redis = new Redis();

	// Add some dummy data

	$redis->rpush('drinks', 'water');
	$redis->rpush('drinks', 'milk');
	$redis->rpush('drinks', 'orange juice');

	// Prints out a list of drinks

	echo HTML::ol($redis->lrange('drinks', 0, -1));

	// Delete data

	$redis->del('drinks');

If the redis command contains spaces (CONFIG GET, CONFIG SET, etc...) then you'll have to separate the words using camel case or underscores.

	// Use camel case to separate multi word commands

	$redis->configGet('*max-*-entries*');

	// You can also use underscores

	$redis->config_get('*max-*-entries*');

There is also a magic shortcut that can be used to run simple commands against the default Redis connection.

	Redis::set('foo', 'bar');

	echo Redis::get('foo');

	Redis::del('foo');