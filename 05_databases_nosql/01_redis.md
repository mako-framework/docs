# Redis

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage:basics)
	- [Magic shortcut](#magic_shortcut)

--------------------------------------------------------

The Redis client provides a simple and consistent way of communicating with a [Redis](http://redis.io) server.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:basics"></a>

#### Basics

Creating a database connection is done using the ```ConnectionManager::connection``` method.

	// Returns connection object using the "default" redis configuration defined in the config file

	$redis = $this->redis->connection();

	// Returns connection object using the "mydb" redis configuration defined in the config file

	$redis = $this->redis->connection('mydb');

The Redis class uses the magic ```__call``` method so every current (and future) [Redis command](http://redis.io/commands) is a valid method.

	// Add some dummy data

	$redis->rpush('drinks', 'water');
	$redis->rpush('drinks', 'milk');
	$redis->rpush('drinks', 'orange juice');

	// Fetches all the drinks

	$drinks = $redis->lrange('drinks', 0, -1);

	// Delete data

	$redis->del('drinks');

If the redis command contains spaces (```CONFIG GET```, ```CONFIG SET```, etc ...) then you'll have to separate the words using camel case or underscores.

	// Use camel case to separate multi word commands

	$redis->configGet('*max-*-entries*');

	// You can also use underscores

	$redis->config_get('*max-*-entries*');

The ```pipeline``` method allows you to send multiple commands to the Redis server without having to wait for the replies. Using pipelining can be useful if you need to send a large number of commands as you will not be paying the cost of round-trip time for every single call.

	$redis->set('x', 0);

	$replies = $redis->pipeline(function($redis)
	{
		for($i = 0; $i < 100; $i++)
		{
			$redis->incr('x');
		}
	});

<a id="usage:magic_shortcut"></a>

#### Magic shortcut

You can access the default redis connection directly without having to go through the ```connection``` method thanks to the magic ```__call``` method.

	$exists = $this->redis->exists('drinks');
