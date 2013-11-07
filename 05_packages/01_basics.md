# Packages

--------------------------------------------------------

* [Basics](#basics)
* [Routing](#routing)
* [Configuration, tasks, urls and views](#configuration_tasks_urls_and_views)
* [Installing packages](#installing_packages)
* [Using packages](#using_packages)

--------------------------------------------------------

Packages can be used to add, extend or override functionality. Packages can contain classes, config files, controllers, language files, migrations, tasks and views.

> Package names must be lower case and can only contain ascii alphabet characters, numbers and underscores.

--------------------------------------------------------

<a id="basics"></a>

### Basics

Packages are installed in the ```app/packages``` directory. Here's the directory structure of a basic package containing only one class:

	app/packages/foo/
		_init.php
		classes/
			Foo.php

All packages must include a ```_init.php``` file. This is where you define constants, global functions and set up autoloading of the package classes. You can use the Mako autoloader or write your own.

	<?php

	// Map classes

	mako\ClassLoader::mapClasses(array
	(
		'Foo' => __DIR__ . '/classes/Foo.php',
	));

> You do not need to map or set up autoloading for controllers or models as this is handled by the framework.

Here's the directory structure of a more complex package containing config files, controllers, language files, migrations, tasks and views.

	app/packages/foobar/
		_init.php
		classes/
			Foo.php
			Bar.php
		config/
			config.php
			routes.php
		controllers/
			index.php
		i18n/
			strings/
				language.php
		tasks/
			ask.php
		views/
			view.php

--------------------------------------------------------

<a id="routing"></a>

### Routing

You can route requests to package controllers by adding your package to the ```packages``` list in the ```app/config/routes.php``` config file. The array key is the base route you want the package to respond to and the value is the package name.

	'packages' => array
	(
		// The foobar package will now handle all requests starting with "my/base/route"
		
		'my/base/route' => 'foobar'
	);

The package will automatically be initialized when a request matches the base route you have defined and the ```<package_name>\controllers``` namespace will be registered by the autoloader allowing your controllers to be autoloaded.

Routable packages also require their own ```routes.php``` config file where you define the default route and any custom routes you might need.

--------------------------------------------------------

<a id="configuration_tasks_urls_and_views"></a>

### Configuration, tasks, urls and views

Loading configuration files in a package is done using the ```Config::get``` method like in the application. The only difference is that you have to prefix the name of the config file with the name of your package followed by two colons (```::```). The same convention is used when running tasks, generating urls or creating views.

	// Loads the "app/packages/foobar/config/config.php" file

	$config = Config::get('foobar::config');

	// Execute the default task action

	php reactor foobar::task

	// Generates a URL for the foobar package where "foobar::" is replaced by the base route

	echo URL::('foobar::users/profile/1');

	// Loads the "app/packages/foobar/views/view.php" file

	$view = new View('foobar::view');

--------------------------------------------------------

<a id="installing_packages"></a>

### Installing packages

Installing packages is extremely easy. Just drop the package in the ```app/packages``` directory and you're ready to go! 

You can also install packages using [composer](http://packagist.org/):

	composer require mako/image:*

--------------------------------------------------------

<a id="using_packages"></a>

### Using packages

You need to initialize a package before you can use it in your application. This can be done using the ```Package::init``` method or by registering the package(s) you want to auto-initialize in the ```app/config/application.php``` config file.

	Package::init('image');

> It is recommended to manually initialize packages when needed if you do not use them throughout your entire application.


