# Packages

--------------------------------------------------------

* [Basics](#basics)
* [Routing](#routing)
* [Configuration, tasks, i18n and views](#configuration_tasks_i18n_and_views)
* [Installing packages](#installing_packages)

--------------------------------------------------------

Packages can be used to add, extend or override functionality. You can look at them as small isolated applications as they can contain classes, config files, controllers, language files, migrations, tasks and views. 

> Packages are installed in the ```app/packages``` directory. Package names should only contain ascii alphabet characters, numbers and underscores.

--------------------------------------------------------

<a id="basics"></a>

### Basics

Here's the directory structure of a basic package containing only a single class.

	app/packages/foo/
		classes/
			Foo.php

Here's the directory structure of a more complex package containing config files, controllers, language files, tasks and views.

	app/packages/foobar/
		classes/
			Foo.php
			Bar.php
		config/
			config.php
		controllers/
			Home.php
		i18n/
			strings/
				en_US/
					language.php
		tasks/
			Task.php
		views/
			welcome.php
		routes.php

> If your package isn't installed via [composer]((:base_url:/docs/:version:/packages:composer-packages)) then you'll have to edit the application [composer.json](https://getcomposer.org/doc/04-schema.md#autoload) file and add the required autoloading mappings there. Remember to run ```composer dump-autoload``` when you're done.

--------------------------------------------------------

<a id="routing"></a>

### Routing

You can create a ```routes.php``` file in your package route directory and define your package routes there.

	$routes->group(['prefix' => 'foobar'], function($routes)
	{
		$routes->get('/', 'foo\controllers\Home::welcome', 'foo:home');
	});

Now all you have to do is include the package routes in your main routes file (```app/routes.php```).

	include __DIR__ . '/packages/foobar/routes.php';

--------------------------------------------------------

<a id="configuration_tasks_i18n_and_views"></a>

### Configuration, tasks, i18n and views

Loading configuration files in packages is a bit different from what you're used to. You have to prefix the name of the config file with the name of your package followed by two colons (```::```). The same convention is used when running tasks, fetching i18n strings or creating views.

	// Loads the "app/packages/foobar/config/config.php" file

	$config = $this->config->get('foobar::config');

	// Execute the default task action

	php reactor foobar::task

	// Fetches a i18n string from "app/packages/foobar/language.php"

	$string = $this->i18n->get('foobar::language.string');

	// Loads the "app/packages/foobar/views/view.php" file

	$view = $this->view->create('foobar::welcome');

--------------------------------------------------------

<a id="installing_packages"></a>

### Installing packages

Installing packages is extremely easy. Just drop the package in the ```app/packages``` directory and you're ready to go! 

You can also install packages using [composer]((:base_url:/docs/:version:/packages:composer-packages).

	composer require <vendor>/<package name>:*

> Remember to include the package routes in your ```app/routes.php``` file.