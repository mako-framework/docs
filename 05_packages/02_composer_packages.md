# Composer packages

--------------------------------------------------------

* [Basics](#basics)
* [Creating packages](#creating_packages)
* [Publishing packages](#publishing_packages)

--------------------------------------------------------

Mako packages can be installed manually and via [composer](http://packagist.org/).

--------------------------------------------------------

<a id="basics"></a>

### Basics

Installing a Mako composer package is easy and only requires one single command.

	composer require <vendor>/<package name>:*

You'll be able to upate your packages with a single command as well.

	composer update

> Package names should only contain ascii alphabet characters, numbers and underscores.

--------------------------------------------------------

<a id="creating_packages"></a>

### Creating composer packages

Mako composer packages require a ```composer.json``` file in the root of the package.

	{
		"name": "foo/foobar",
		"type": "mako-package",
		"autoload": {
			"psr-4": {
				"foobar\\": ""
			}
		},
		"require": {
			"composer/installers": "*"
		}
	}

You'll notice that the type is set to ```mako-package``` and that the ```composer/installers``` package is required. This ensures that the packages are installed in the ```app/packages``` directory.

You can read more about composer.json files at [getcomposer.org](https://getcomposer.org/doc/04-schema.md).

--------------------------------------------------------

<a id="publishing_packages"></a>

### Publishing packages

Read about how to publish a package at [packagist.org](http://packagist.org/).