# Upgrading

--------------------------------------------------------

* [Application](#application)
* [Framework](#framework)
	- [Request](#framework:request)
	- [Response](#framework:response)
	- [Commands](#framework:commands)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```5.3.x``` to ```5.4.x```.

--------------------------------------------------------

<a id="application"></a>

### Application

The `MAKO_START` constant has been removed. Use the `Application::startTime()` method instead.

--------------------------------------------------------

<a id="framework"></a>

### Framework

<a id="framework:request"></a>

#### Request

All methods that were deprecated in 5.3 have been removed. Check out the [request](:base_url:/docs/:version:/routing-and-controllers:request) docs for information on how to upgrade your application.

<a id="framework:response"></a>

#### Response

Response filters have been removed. [Middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware) should be used to achieve the same results.

<a id="framework:commands"></a>

#### Commands

Parameters passed to the `execute` method of reactor commands are now converted to camel case.

	// Before: php app/reactor command --cache-path=/foo/bar

	public function execute($cache_path)
	{
		// ....
	}

	// Now: php app/reactor command --cache-path=/foo/bar

	public function execute($cachePath)
	{
		// ....
	}
