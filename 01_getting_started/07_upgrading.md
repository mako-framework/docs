# Upgrading

--------------------------------------------------------

* [4.2.x to 4.3.x](#4.2.x_to_4.3.x)
	- [Application](#application)
		- [Route parameters](#application:route_parameters)
		- [Route constraints](#application:route_constraints)
		- [HTTP exceptions](#application:http_exceptions)
	- [Packages](#packages)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```4.2.x``` to ```4.3.x```.

--------------------------------------------------------

<a id="4.2.x_to_4.3.x"></a>

### 4.2.x to 4.3.x

<a id="application"></a>

#### Application

<a id="application:route_parameters"></a>

##### Route parameters

Route parameters must now have the same name as your route action parameters. So if you have the following route:

	$routes->get('/article/{id}', 'Articles::view');

Then your action will have to have a parameter named ```$id``` for it to work:

	public function view($id)
	{
		return $id;
	}

<a id="application:route_constraints"></a>

##### Route constraints

The ```constraints``` method has been renamed to ```when```:

	$routes->get('/article/{id}', 'Articles::view')->when(['id' => '[0-9]+']);

<a id="application:http_exceptions"></a>

##### HTTP exceptions

All the HTTP exceptions have been moved to the ```mako\http\exceptions``` namespace and the ```PageNotFoundException``` has been renamed to ```NotFoundException```.

--------------------------------------------------------

<a id="application"></a>

#### Packages

The changes above will also affect packages with routes and/or HTTP exceptions.