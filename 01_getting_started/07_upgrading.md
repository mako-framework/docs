# Upgrading

--------------------------------------------------------

* [Application](#application)
	- [Configuration](#application:configuration)
	- [Command line](#application:command-line)
	- [Migrations](#application:migrations)
	- [Miscellaneous](#application:miscellaneous)
* [Packages](#packages)
	- [Command line](#packages:command-line)
	- [Migrations](#packages:migrations)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```4.3.x``` to ```4.4.x```.

--------------------------------------------------------


<a id="application"></a>

### Application

<a id="application:configuration"></a>

#### Configuration

The ```app/config/application.php``` and ```app/config/session.php``` configuration files have seen some minor changes:

* [application.php](https://github.com/mako-framework/app/blob/23cb6cdf3ccce2d686a69f45cf0ee6cd648b7207/app/config/application.php)
* [session.php](https://github.com/mako-framework/app/blob/bbf29d5fad11cd0bc9d8a1434d53f3eb03b10ed1/app/config/session.php)
 
<a id="application:command-line"></a>

#### Command line

The Mako command line tool ```reactor``` has been rewritten from scratch. Check out the [documentation](http://localhost:8002/mako/docs/public/index.php/docs/4.4/command-line:basics) to see what you'll have to do to migrate your code.

<a id="application:migrations"></a>

#### Migrations

Migrations classes must now extend the ```mako\database\migrations\Migration``` class.

<a id="application:miscellaneous"></a>

#### Miscellaneous

* The ```init.php``` file has been moved from the framework core to the [application root directory](https://github.com/mako-framework/app/blob/5bfb27b6e22cb87c088cc0bc56d1328c15f34953/app/init.php).
* The ```reactor``` file has been [updated](https://github.com/mako-framework/app/blob/a158f548542ddce065726149a0e96302250cb372/app/reactor).
* The ```index.php``` file has been [updated](https://github.com/mako-framework/app/blob/a158f548542ddce065726149a0e96302250cb372/public/index.php).

> The default application structure has been slightly [changed](https://github.com/mako-framework/app/tree/a158f548542ddce065726149a0e96302250cb372).

--------------------------------------------------------

<a id="packages"></a>

### Packages

<a id="packages:command-line"></a>

#### Command line

The Mako command line tool ```reactor``` has been rewritten from scratch. Check out the [documentation](http://localhost:8002/mako/docs/public/index.php/docs/4.4/command-line:basics) to see what you'll have to do to migrate your code.

Also check out the [package documentation](:base_url:/docs/:version:/packages:packages) to see how your register your package commands.

<a id="packages:migrations"></a>

#### Migrations

Migrations classes must now extend the ```mako\database\migrations\Migration``` class.