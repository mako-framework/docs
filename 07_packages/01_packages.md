# Packages

--------------------------------------------------------

* [Basics](#basics)
* [Configuration, i18n and views](#configuration_i18n_and_views)
	- [Overriding configuration, i18n and views](#configuration_i18n_and_views:overriding_configuration_i18n_and_views)
* [Commands](#commands)
* [Package installation](#package_installation)
* [Publishing packages](#publishing_packages)

--------------------------------------------------------

Packages are installed using [composer](https://packagist.org) and can be used to add, extend or override functionality.

--------------------------------------------------------

### <a id="basics" href="#basics">Basics</a>

There are two files that must be present in a package. A package "boot" class (class must extend the `mako\application\Package` class) and a composer.json file.

```
├─ src/
│  └─ FooPackage.php
└─ composer.json
```
{.language-none}

The package class is used to register your package with the application that uses it. The minimum information you need to provide is the package name.

```
<?php

namespace acme\foo;

use mako\application\Package;

class FooPackage extends Package
{
	protected $packageName = 'acme/foo';
}
```

You also have to set the name of your package in the composer.json file. This is also where you define the type to autoloading to use when loading classes from your package (PSR-4 is recommended).

The example file below only contains the bare minimum so head over to the [composer website](https://getcomposer.org) for a full overview of the complete composer.json schema.

```
{
	"name": "acme/foo",
	"autoload": {
		"psr-4": {
			"acme\\foo\\": "src"
		}
	}
}
```
{.language-json}

--------------------------------------------------------

### <a id="configuration_i18n_and_views" href="#configuration_i18n_and_views">Configuration, i18n and views</a>

Packages can also have their own configuration files, language strings and views. The directory tree below shows you the default directory structure but you can change it if you want to. All you have to do is override the appropriate `get*Path` method in your package class.

```
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
```
{.language-none}

Loading configuration files, language strings and views from a package is a little different from loading them from an application. You have to prefix the name of the item you want to load with the package file namespace followed by to colons (`::`).

```
$view = $this->view->create('acme-foo::vista');
```

By default the file namespace of a package will be the package name where the slash has been replaced by a hyphen (e.g. `acme/foo` becomes `acme-foo`). You can override this by setting the `$fileNamespace` property in your package class.

```
protected $fileNamespace = 'foo';
```

#### <a id="configuration_i18n_and_views:overriding_configuration_i18n_and_views" href="#configuration_i18n_and_views:overriding_configuration_i18n_and_views">Overriding configuration, i18n and views</a>

Sometimes you might want to modify the configuration, i18n strings or views of a third party package. You can edit the them directly but the changes you make will be overwritten when you update the package. This is where the cascading file lookup comes in handy.

Lets say you have a packaged named `acme-foo` with a config file you want to modify. Just copy the file into `app/config/packages/acme-foo` and the application will load your copy instead of the one located in the package. This makes it possible to update the package while keeping your custom settings. The same convention also works for i18n and view files.

> You can also override the default Mako error views using this method. Just create a `app/resources/views/packages/mako-error` directory and add a file named `404.tpl.php` to create your own custom 404 template.

--------------------------------------------------------

### <a id="commands" href="#commands">Commands</a>

Registering [package](:base_url:/docs/:version:/command-line:commands) commands is done using the `$commands` property in your package class.

```
protected $commands =
[
	'acme-foo::command' => acme\foo\commands\Command::class,
];
```

> It is considered good practice to prefix the command with your package prefix to avoid naming collisions with other commands.

--------------------------------------------------------

### <a id="package_installation" href="#package_installation">Package installation</a>

Installing packages is extremely easy. All you need to do is running a simple [composer](https://getcomposer.org) command and add the package "boot" class to the list of packages in your `app/config/application.php` configuration file.

```
composer require <vendor>/<package name>
```
{.language-none}

So, to install `acme/foo` package using composer, you have to issue following command from the project directory

```
composer require acme/foo
```
{.language-none}

--------------------------------------------------------

### <a id="publishing_packages" href="#publishing_packages">Publishing packages</a>

Publishing your packages is very easy and you can read all about it on the [packagist.org](https://packagist.org) website.

You can also host your own private repository if you don't want your code to be publicly available. You can read more about how to set up and manage your own repository on the [getcomposer.org](https://getcomposer.org/doc/05-repositories.md#hosting-your-own) website.
