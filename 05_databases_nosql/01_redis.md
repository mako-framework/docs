# Redis

--------------------------------------------------------

* [Connections](#connections)
	- [Basics](#connections:basics)
	- [Magic shortcut](#connections:magic_shortcut)
* [Commands](#commands)
* [Pipelining](#pipelining)
* [Pub/Sub](#pub_sub)
	- [Publishing](#pub_sub:publishing)
	- [Subscribing](#pub_sub:subscribing)
* [Monitor](#monitor)

--------------------------------------------------------

The Redis client provides a simple and consistent way of communicating with a [Redis](https://redis.io) server.

--------------------------------------------------------

### <a id="connections" href="#connections">Connections</a>

#### <a id="connections:basics" href="#connections:basics">Basics</a>

Creating a database connection is done using the `ConnectionManager::getConnection()` method.

```
// Returns connection object using the "default" redis configuration defined in the config file

$redis = $this->redis->getConnection();

// Returns connection object using the "mydb" redis configuration defined in the config file

$redis = $this->redis->getConnection('mydb');
```

#### <a id="connections:magic_shortcut" href="#connections:magic_shortcut">Magic shortcut</a>

You can access the default redis connection directly without having to go through the `getConnection` method thanks to the magic `__call` method.

```
$lolWut = $this->redis->lolWut(); // Yes, this is a valid Redis command
```

--------------------------------------------------------

### <a id="commands" href="#commands">Commands</a>

The Redis client supports all official [commands](https://redis.io/docs/latest/commands/) including the ones included in the Redis Stack (Bloom filter, JSON, Redis Query Engine, etc...) as well as RESP2 and RESP3 responses.

```
// Add some dummy data

$redis->rPush('drinks', 'water');
$redis->rPush('drinks', 'milk');
$redis->rPush('drinks', 'orange juice');

// Fetch all the drinks

$drinks = $redis->lRange('drinks', 0, -1);

// Delete data

$redis->del('drinks');
```

If you encounter a new command that hasn't been implemented in the client yet then you can use the `executeCommand` method. This can also be useful when working with a Redis compatible superset like [KeyDB](https://docs.keydb.dev/).

```
// Add some dummy data

$redis->executeCommand('RPUSH', 'drinks', 'water');
$redis->executeCommand('RPUSH', 'drinks', 'milk');
$redis->executeCommand('RPUSH', 'drinks', 'orange juice');

// Fetch all the drinks

$drinks = $redis->executeCommand('LRANGE', 'drinks', 0, -1);

// Delete data

$redis->executeCommand('DEL', 'drinks');
```

--------------------------------------------------------

### <a id="pipelining" href="#pipelining">Pipelining</a>

The `pipeline` method allows you to send multiple commands to the Redis server without having to wait for the replies. Using pipelining can be useful if you need to send a large number of commands as you will not be paying the cost of round-trip time for every single call.

```
$redis->set('x', 0);

$replies = $redis->pipeline(function ($redis) {
	for ($i = 0; $i < 100; $i++) {
		$redis->incr('x');
	}
});
```

> Note: Pipelining will not work when connected to a Redis cluster unless all keys used in the pipeline are stored on the same node.
{.warning}

--------------------------------------------------------

### <a id="pub_sub" href="#pub_sub">Pub/Sub</a>

#### <a id="pub_sub:publishing" href="#pub_sub:publishing">Publishing</a>

You can publish messages to channels using the `publish` method. The first parameter is the channel name while the second parameter is your message. The method will return the number of subscribers that have received the message.

```
$redis->publish('channel1', 'Hello, World!');
```

#### <a id="pub_sub:subscribing" href="#pub_sub:subscribing">Subscribing</a>

The `subscribeTo` method allows you to subscribe to channels. You can also use the `subscribeToPattern` method if you want to subscribe using [channel name patterns](https://redis.io/commands/psubscribe).

> Message subscribers are blocking and should *not* be used in controllers. [Reactor commands](:base_url:/docs/:version:/command-line:commands), however, are perfect for handling long running tasks. You should also set the `timeout` value of the connection used to subscribe to `-1` in your redis configuration file to avoid dropped connections while waiting for new messages.

```
$redis->subscribeTo(['channel1', 'channel2'], function ($message) {
	$this->write($message);
});
```

By default the subscribers will only receive messages (`message` and `pmessage`). You can however subscribe to more messages types using the optional third parameter.

```
$redis->subscribeTo(['channel1', 'channel2'], function ($message) {
	$this->write($message);
}, ['message', 'subscribe', 'pong']);
```

You can stop subscribing at any time by returning `false` from the subscriber closure.

The message passed to the subscriber is an instance of the `Message` object. It implements the `__toString` method so you can treat it like a string but there are also a set of methods available if you want to access some additional information about the message.

| Method       | Description                                      |
|--------------|--------------------------------------------------|
| getType()    | Returns the message type                         |
| getChannel() | Returns the channel name the message was sent to |
| getPattern() | Returns the channel pattern that was matched     |
| getBody()    | Returns the message body                         |

--------------------------------------------------------

### <a id="monitor" href="#monitor">Monitor</a>

The `monitor` method can be useful when debugging applications that use Redis. Once called the server will stream back every command processed by the server.

> The method should only be used in a [reactor command](:base_url:/docs/:version:/command-line:commands). You should also set the `timeout` value of the connection used to monitor to `-1` in your redis configuration file to avoid dropped connections while waiting for new commands.

```
$redis->monitor(function ($command) {
	$this->write($command);
});
```

You can stop the monitor at any time by returning `false` from your monitor closure.

> Note: The connection is closed as soon as the monitor closure returns `false`.