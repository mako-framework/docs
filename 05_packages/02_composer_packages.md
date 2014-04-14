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

Installing a Mako package using composer is easy and only requires a single command.

	composer require <vendor>/<package name>:*

Another advantage is that you'll now be able to upate your packages with a single command as well.

	composer update

--------------------------------------------------------

<a id="creating_packages"></a>

### Creating packages

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

> Package names should only contain ascii alphabet characters, numbers and underscores.

You can read more about the composer.json schema at [getcomposer.org](https://getcomposer.org/doc/04-schema.md).

--------------------------------------------------------

<a id="publishing_packages"></a>

### Publishing packages

Publishing your packages is very easy. Read all about it at [packagist.org](http://packagist.org/). 

You can also host your own private repository if you don't want your code to be avaialbe to the public, read more about it at [getcomposer.org](https://getcomposer.org/doc/05-repositories.md#hosting-your-own).