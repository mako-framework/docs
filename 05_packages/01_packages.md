#Packages

--------------------------------------------------------

* [Basics](#basics)
* [Configuration, i18n and views](#configuration_i18n_and_views)
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

	use \mako\application\Package;

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

Package can also have their own configuration files, language strings and views. The directory tree below shows you the default directory structure but you can change it if you want to. All you have to do is override the appropriate ```get*Path``` method in your package class.

 	├─ config/
 	|  └─ ...
 	├─ i18n/
	|  └─ en_US/
	|     └─ strings/
    |        └─ ...
	├─ src/
 	│  └─ FooPackage.php
 	├─ views/
 	|  └─ ...
	└─ composer.json

Loading configuration files, language strings and views from a package is a little different from loading them from an application. You have to prefix the name of the item you want to load by the package file namespace followed by to colons (```::```).

	$view = $this->view->create('acme-foo::hello');

By default the file namespace of a package will be the package name where the slash has been replaced by a hyphen (e.g. ```acme/foo``` becomes ```acme-foo```). You can override this by setting the ```$fileNamespace``` in your package class.

	protected $fileNamespace = 'foo';

--------------------------------------------------------

<a id="package_installation"></a>

### Package installation

Installing packages is extremely easy. All you need to do is run a simple [composer](http://localhost:8002/mako/docs/docs/4.0/packages:composer-packages) command and add the package "boot" class to the list of packages in your ```app/config/application.php``` configuration file.

	composer require acme/foo:*

--------------------------------------------------------

<a id="publishing_packages"></a>

### Publishing packages

Publishing your packages is very easy. Read all about it at [packagist.org](http://packagist.org/).

You can also host your own private repository if you don't want your code to be publicly avaialbe. You can read more about how to set up and manage your own repository at [getcomposer.org](https://getcomposer.org/doc/05-repositories.md#hosting-your-own).