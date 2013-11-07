# Autoloading

--------------------------------------------------------

* [Usage](#usage)
* [Class aliases](#class_aliases)

--------------------------------------------------------

The ```ClassLoader``` class handles autoloading of classes in the Mako framework.

You'll normally not have to worry about class autoloading as this is done automatically by the framework. You can in some cases optimize class loading by running the following ```composer``` command:

	composer dump-autoload -o

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```mapClass``` method will add a class to the autoloader mappings.

	ClassLoader::mapClass('FooBar', '/path/to/FooBar.php');

	ClassLoader::mapClass('foo\FooBar', '/path/to/foo/FooBar.php');

The ```mapClasses``` method will add an array of classes to the autoloader mappings.

	ClassLoader::mapClasses(array
	(
		'Foo' => '/path/to/Foo.php',
		'Bar' => '/path/to/Bar.php',
	));

The ```directory``` method will add an additional path from which the autoloader will try to load PSR-0 compatible classes.

	ClassLoader::directory('/path/to/library');

The ```registerNamespace``` method allows you to register a PSR-0 compatible namespace with the autoloader.

	ClassLoader::registerNamespace('Zend\XmlRpc', MAKO_LIBRARIES_PATH . '/XmlRpc');

--------------------------------------------------------

<a id="class_aliases"></a>

### Class aliases

You can register class aliases in the ```app/config/application.php``` config file. The key is the alias while the value is the actual class name.

	'aliases' => array
	(
		'URL'    => 'mako\URL',
		'Assets' => 'mako\Assets',
	),