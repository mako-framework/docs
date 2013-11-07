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

Installing a Mako package using composer is easy and only requires one single command:

	composer require mako/image:*

Using composer makes it easy to manage dependencies between packages and upgrading packages can be done with one simple command in the CLI.

	composer update

> Package names must be lower case and can only contain ascii alphabet characters, numbers and underscores.

--------------------------------------------------------

<a id="creating_packages"></a>

### Creating packages

Mako composer packages require a ```composer.json``` file in the root of the package. Here's what a minimal ```composer.json``` file for a Mako package looks like:

	{
		"name": "vendor-name/package_name",
		"type": "mako-package",
		"require": {
			"composer/installers": "*"
		}
	}

Here's what the ```composer.json``` file of a Twig view renderer package looks like:

	{
		"name": "kula-shakerz/twig_renderer",
		"type": "mako-package",
		"description": "Twig view renderer for the Mako framework",
		"keywords": ["mako", "twig"],
		"homepage": "https://github.com/kula-shakerz",
		"license": "BSD-3-Clause",
		"authors": [
			{
				"name": "Frederic G. Østby",
				"email": "frederic.g.ostby@gmail.com"
			}
		],
		"require": {
			"php": ">=5.3.1",
			"composer/installers": "*",
			"twig/twig": "1.*"
		}
	}

You'll notice that the type is set to ```mako-package``` and that the ```composer/installers``` package is required in both examples. This ensures that the packages will be installed in the ```app/packages``` directory.

> Adding ```mako``` to the list of keywords makes it easier to find Mako packages in the packagist repository.

You can read more about composer at [getcomposer.org](http://getcomposer.org/).

--------------------------------------------------------

<a id="publishing_packages"></a>

### Publishing packages

Read about how to publish a package at [packagist.org](http://packagist.org/).