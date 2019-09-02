# Redis

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage:basics)
	- [Pipelining](#usage:pipelining)
	- [Pub/Sub](#usage:pub_sub)
		- [Publishing](#usage:pub_sub:publishing)
		- [Subscribing](#usage:pub_sub:subscribing)
	- [Magic shortcut](#usage:magic_shortcut)

--------------------------------------------------------

The Redis client provides a simple and consistent way of communicating with a [Redis](http://redis.io) server.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:basics"></a>

#### Basics

Creating a database connection is done using the `ConnectionManager::connection()` method.

```
// Returns connection object using the "default" redis configuration defined in the config file

$redis = $this->redis->connection();

// Returns connection object using the "mydb" redis configuration defined in the config file

$redis = $this->redis->connection('mydb');
```

The Redis client uses the magic `__call` method to build commands method so every current (and future) [Redis command](http://redis.io/commands) is a valid method.

```
// Add some dummy data

$redis->rpush('drinks', 'water');
$redis->rpush('drinks', 'milk');
$redis->rpush('drinks', 'orange juice');

// Fetches all the drinks

$drinks = $redis->lrange('drinks', 0, -1);

// Delete data

$redis->del('drinks');
```

If the redis command contains spaces (`CONFIG GET`, `CONFIG SET`, etc ...) then you'll have to separate the words using camel case or underscores.

```
// Use camel case to separate multi word commands

$redis->configGet('*max-*-entries*');

// You can also use underscores

$redis->config_get('*max-*-entries*');
```

<a id="usage:pipelining"></a>

#### Pipelining

The `pipeline` method allows you to send multiple commands to the Redis server without having to wait for the replies. Using pipelining can be useful if you need to send a large number of commands as you will not be paying the cost of round-trip time for every single call.

```
$redis->set('x', 0);

$replies = $redis->pipeline(function($redis)
{
	for($i = 0; $i < 100; $i++)
	{
		$redis->incr('x');
	}
});
```

> Note that pipelining will not work when connected to a Redis cluster unless all keys used in the pipeline are stored on the same node.
{.warning}

<a id="usage:pub_sub"></a>

#### Pub/Sub

<a id="usage:pub_sub:publishing"></a>

##### Publishing

You can publish messages to channels using the `publish` method. The first parameter is the channel name while the second parameter is your message. The method will return the number of subscribers that have received the message.

```
$this->redis->publish('channel1', 'Hello, World!');
```

<a id="usage:pub_sub:subscribing"></a>

##### Subscribing

The `subscribeTo` method allows you to subscribe to channels. You can also use the `subscribeToPattern` method if you want to subscribe using [channel name patterns](https://redis.io/commands/psubscribe).

> Message subscribers are blocking and should *not* be used in controllers. Command line [commands](:base_url:/docs/:version:/command-line:commands), however, are perfect for handling long running tasks.

```
$redis->subscribeTo(['channel1', 'channel2'], function($message)
{
	$this->write($message);
});
```

By default the subscribers will only receive messages (`message` and `pmessage`). You can however subscribe to more messages types using the optional third parameter.

```
$redis->subscribeTo(['channel1', 'channel2'], function($message)
{
	$this->write($message);
}, ['message', 'subscribe', 'pong']);
```

You can stop subscribing at any time by returning `false` from the subscriber closure.

The message passed to the subscriber is an instance of the `Message` object. It implements the `__toString` method so you can treat it like a string but there are also a set of methods available if you want to access some additional information about the message.

| Method       | Description                                     |
|--------------|-------------------------------------------------|
| getType()    | Returns the message type                        |
| getChannel() | Returns the channel name the message was set to |
| getPattern() | Returns the channel pattern that was matched    |
| getBody()    | Returns the message body                        |

<a id="usage:magic_shortcut"></a>

#### Magic shortcut

You can access the default redis connection directly without having to go through the `connection` method thanks to the magic `__call` method.

```
$exists = $this->redis->exists('drinks');
```
