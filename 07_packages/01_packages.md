#Packages

--------------------------------------------------------

* [Basics](#basics)
* [Configuration, i18n and views](#configuration_i18n_and_views)
* [Commands](#commands)
* [Package installation](#package_installation)
* [Publishing packages](#publishing_packages)

--------------------------------------------------------

Packages are installed using [composer](http://packagist.org/) and can be used to add, extend or override functionality.

--------------------------------------------------------

<a id="basics"></a>

### Basics

There are two files that must be present in a package. A package "boot" class (class must extend the ```mako\application\Package``` class) and a composer.json file.

	├─ src/
 	│  └─ FooPackage.php
	└─ composer.json

The package class is used to register your package with the application that uses it. The minimum information you need to provide is the package name.

	<?php

	namespace acme\foo;

	use mako\application\Package;

	class FooPackage extends Package
	{
		protected $packageName = 'acme/foo';
	}

You also have to set the name of your package in the composer.json file. This is also where you define the type to autoloading to use when loading classes from your package (PSR-4 is recommended).

The example file below only contains the bare minimum so head over to the [composer website](https://getcomposer.org/) for a full overview of the complete composer.json schema.

	{
		"name": "acme/foo",
		"autoload": {
			"psr-4": {
				"acme\\foo\\": "src"
			}
		}
	}

--------------------------------------------------------

<a id="configuration_i18n_and_views"></a>

### Configuration, i18n and views

Packages can also have their own configuration files, language strings and views. The directory tree below shows you the default directory structure but you can change it if you want to. All you have to do is override the appropriate ```get*Path``` method in your package class.

	├─ config/
 	|  └─ ...
 	├─ resources/
 	|  ├─ i18n/
	|  |  └─ en_US/
	|  |     └─ strings/
	|  |        └─ ...
	|  └─ views/
	|     └─ ...
	├─ src/
 	│  └─ FooPackage.php
	└─ composer.json

Loading configuration files, language strings and views from a package is a little different from loading them from an application. You have to prefix the name of the item you want to load with the package file namespace followed by to colons (```::```).

	$view = $this->view->create('acme-foo::vista');

By default the file namespace of a package will be the package name where the slash has been replaced by a hyphen (e.g. ```acme/foo``` becomes ```acme-foo```). You can override this by setting the ```$fileNamespace``` property in your package class.

	protected $fileNamespace = 'foo';

--------------------------------------------------------

<a id="commands"></a>

### Commands

Registering [package](:base_url:/docs/:version:/command-line:custom-commands) commands is done using the ```$commands``` property in your package class.

	protected $commands =
	[
		'acme-foo::command' => 'acme\foo\commands\Command',
	];

> It is considered good practice to prefix the command with your package prefix to avoid naming collisions with other commands.

--------------------------------------------------------

<a id="package_installation"></a>

### Package installation

Installing packages is extremely easy. All you need to do is running a simple [composer](https://getcomposer.org/) command and add the package "boot" class to the list of packages in your ```app/config/application.php``` configuration file.

	composer require <vendor>/<package name>:*

So, to install ```acme/foo``` package using composer, you have to issue following command from the project directory

	composer require acme/foo:*

--------------------------------------------------------

<a id="publishing_packages"></a>

### Publishing packages

Publishing your packages is very easy and you can read all about it on the [packagist.org](http://packagist.org/) website.

You can also host your own private repository if you don't want your code to be publicly available. You can read more about how to set up and manage your own repository on the [getcomposer.org](https://getcomposer.org/doc/05-repositories.md#hosting-your-own) website.
