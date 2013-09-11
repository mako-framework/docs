# Request

--------------------------------------------------------

* [Usage](#usage)
* [Routing](#routing)

--------------------------------------------------------

The task of the request class is to gather information about the current request and route requests to the appropriate action.

An instance of the request object is always available in controllers and it can be accessed like this:

	$this->request

--------------------------------------------------------

<a id="usage"></a>

### Usage

You can request a controller action within another controller action using subrequests.

	$response = Request::factory('controller/action')->execute();

The ```controller``` method returns the name of the requested controller.

	$controllerName = $this->request->controller();

The ```action``` method returns the name of the requested controller.

	$actionName = $this->request->action();

The ```ip``` method returns the ip address of the client who made the request.

	$ip = $this->request->ip();

The ```referer``` method returns the URL of the referer.

	$referer = $this->request->referer();

The ```route``` method returns the route of the request.

	$route = $this->request->route();

The ```method``` method returns the request method that was used.

	if($this->request->method() == 'POST')
	{
		// Do something
	}

The ```header``` method returns a HTTP header.

	$header = $this->request->header('user-agent');

	// You can also specify a default return value if the header isn't set

	$header = $this->request->header('foo-bar', 'default value');

The ```username``` method returns the basic HTTP authentication username or NULL.

	$username = $this->request->username();

The ```password``` method returns the basic HTTP authentication password or NULL.

	$password = $this->request->password();

The ```isAjax``` method returns TRUE if the request was made using AJAX and FALSE it not.

	if($this->request->isAjax())
	{
		// Do something
	}

The ```isSecure``` method returns TRUE if the request was made using HTTPS and FALSE it not.

	if($this->request->isSecure())
	{
		// Do something
	}

The ```isMain``` method returns TRUE this is is the main request and FALSE if its a subrequest.

	if($this->request->isMain())
	{
		// Do something
	}

The ```main``` method returns an instance of the main request.

	$request = Request::main();

--------------------------------------------------------

<a id="routing"></a>

### Routing

All routing configuration is done in the ```app/config/routes.php``` file.

	'custom_routes' => array
	(
		'hello' => 'foo/bar',
	),

If a user visits ```http://example.org/hello``` then the request will be routed to ```foo/bar``` and the ```bar``` action of the ```foo``` controller will be executed.

Routing can also be a good way of prettifying your URLs. This example shows how you can hide the index action from your users.

	'custom_routes' => array
	(
		'product/([0-9]+)' => 'product/index/$1',
	),

Visiting ```http://example.org/product/10``` will now be the same as if a user went to ```http://example.org/product/index/10```.

> You can define as many custom routes as you want but note that they will be checked in the order that they are defined.