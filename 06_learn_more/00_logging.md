# Logging

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The log class provides a simple and consistent interface to the most common (and some uncommon) logging solutions:

* DebugToolbar
* File
* FirePHP
* Syslog

--------------------------------------------------------

<a id="usage"></a>

### Usage

To get an instance of the default logger just call the ```instance``` method of the log class. If you want to get an instance of any of the other logger configurations defined in the config file then you'll have to pass the configuration name as the first argument to the instance method.

	// Returns instance of the "default" logger configuration defined in the config file

	$log = Log::instance();

	// Returns instance of the "syslog" log configuration defined in the config file

	$log = Log::instance('syslog');

To write something to the log use the ```write``` method. The method returns TRUE on success or FALSE on failure.

	// Writes "Hello, world!" to the log

	$log->write('Hello, world!');

	// Writes "Hello, world!" to the log treating it as a debug message

	$log->write('Hello, world!', Log::DEBUG);

	// You can also use the magic shortcut methods for writing to the default instance

	Log::debug('Hello, world!');

The available log types are ```Log::EMERGENCY```, ```Log::ALERT```, ```Log::CRITICAL```, ```Log::ERROR```, ```Log::WARNING```, ```Log::NOTICE```, ```Log::INFO``` and ```Log::DEBUG```.