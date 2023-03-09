# OpenAPI

--------------------------------------------------------

* [Installation](#installation)
* [Usage](#usage)

--------------------------------------------------------

The `mako/open-api` package allows you to generate a [OpenApi](https://www.openapis.org) specification using [attributes](https://www.php.net/manual/en/language.attributes.php) or docblock annotations and to generate routes based on the specification.

--------------------------------------------------------

## <a id="installation" href="#installation">Installation</a>

First you'll need to install the package as a dependency to your project.

```
composer require mako/open-api
```

Next you'll need to add the package to the `cli` section of the `packages` configuration array in the `application.php` config file.

```
'cli' =>
[
	mako\openapi\OpenApiPackage::class,
]
```

And finally replace the contents of your `routes.php` file with the following:

```
<?php

use mako\openapi\generators\routing\Runtime;

if(file_exists(__DIR__ . '/openapi.php'))
{
	include __DIR__ . '/openapi.php';
}
else
{
	(new Runtime($routes))->generateFromYamlFile(dirname(MAKO_APPLICATION_PATH) . '/openapi.yml');
}
```

--------------------------------------------------------

## <a id="usage" href="#usage">Usage</a>

You can build an OpenApi specification using a tool like [Apicurito](https://www.apicur.io/apicurito/pwa/) or by documenting your code using attributes or docblock annotations. To check out the syntax head over to the [zircote/swagger-php](https://github.com/zircote/swagger-php) documentation.

If you want to generate a specification file based on your documentation then you can do so by running the `open-api:generate-spec` command.

To generate a cached route file for production then you'll have to run the `open-api:generate-routes` command.

> Note that you have to set the `operationId` parameter to the fully qualified method name (e.g. app\controllers\Index::welcome) of your controller action for the route generator to work.

```
openapi: 3.0.2
info:
    title: Mako
    version: 1.0.0
    description: Mako example application.
paths:
    '/':
        summary: Displays a welcome page.
        get:
            operationId: 'app\controllers\Index::welcome'
```