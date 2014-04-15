# Logging

--------------------------------------------------------

* [Basics](#basics)
* [Advanced usage](#advanced_usage)

--------------------------------------------------------

Mako uses the [Monolog library](https://github.com/Seldaek/monolog) for logging.

--------------------------------------------------------

<a id="basics"></a>

### Basics

The ```debug``` method is used to log debug data.

	$this->logger->debug('foobar');

The ```info``` method is used to log interesting events.

	$this->logger->info('foobar');

The ```notice``` method is used to log normal but significant events.

	$this->logger->notice('foobar');

The ```warning``` method is used to log exceptional occurrences that are not errors.

	$this->logger->warning('foobar');

The ```error``` method is used to log runtime errors that do not require immediate action but should typically be logged and monitored.

	$this->logger->error('foobar');

The ```critical``` method is used log to critical conditions.

	$this->logger->critical('foobar');

The ```alert``` method is used to log events where action must be taken immediately.

	$this->logger->alert('foobar');

The ```emergency``` method is used to log events where the system is unusable.

	$this->logger->alert('foobar');

--------------------------------------------------------

<a id="advanced_usage"></a>

### Advanced usage

Check out the [Monolog documentation](https://github.com/Seldaek/monolog) for the full documentation of the library. 