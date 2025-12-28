# OpenAPI

--------------------------------------------------------

* [Installation](#installation)
* [Usage](#usage)

--------------------------------------------------------

The `mako/open-api` package allows you to generate an [OpenApi](https://www.openapis.org) specification by documenting your API routes using PHP [attributes](https://www.php.net/manual/en/language.attributes.php), and to generate routes from an OpenAPI specification.

In addition, the package can render interactive API documentation for your OpenAPI specification using one of the following user interface providers:

* Elements
* Redoc
* Scalar
* Swagger (default)

--------------------------------------------------------

### <a id="installation" href="#installation">Installation</a>

First you'll need to install the package as a dependency to your project.

```
composer require mako/open-api
```

Next you'll need to add the package to the `cli` section of the `packages` configuration array in the `application.php` config file.

```
'cli' => [
	mako\openapi\OpenApiPackage::class,
]
```

And finally replace the contents of your `routes.php` file with the following:

```
<?php

use mako\openapi\http\routing\Registrar;

Registrar::register(
	$routes,
	cachedRoutes: __DIR__ . '/openapi.php',
	openApiSpec: __DIR__ . '/openapi.yaml',
);
```

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

You can build an OpenAPI specification using a tool such as [Apicurito](https://www.apicur.io/apicurito/pwa/) or by documenting your code using PHP attributes. For details on the available syntax, see the [zircote/swagger-php](https://github.com/zircote/swagger-php) documentation.

Here is a basic example of documenting API routes using attributes::

```
<?php

namespace app\http\controllers;

use OpenApi\Attributes as OA;

#[OA\Info(
	title: 'Example API',
	version: '1.0.0'
)]
class OpenApiExample
{
	#[OA\Get(
        path: '/',
        responses: [
            new OA\Response(response: 200, description: 'Displays a greeting to the user.'),
        ]
    )]
	public function __invoke(): string
	{
		return 'Welcome to the OpenApi example!';
	}
}
```

If you want to generate a specification file from your documentation, you can do so by running the `open-api:generate-spec` command.

Below is an example of the generated specification:

```
openapi: 3.0.0
info:
  title: 'Example API'
  version: 1.0.0
paths:
  /:
    get:
      operationId: app\http\controllers\OpenApiExample
      responses:
        '200':
          description: 'Displays a greeting to the user.'
```
{.language-yaml}

> Note: If you manually write your OpenAPI specification, you must set the `operationId` parameter to the fully qualified method name (for example, app\controllers\Index::welcome) of the controller action in order for the route generator to work.

When running in production, it is strongly recommended to generate a cached route file for maximum performance. To do so, run the `open-api:generate-routes` command.