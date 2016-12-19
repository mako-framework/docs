# Response

--------------------------------------------------------

* [Basics](#basics)
* [Setting cookies](#setting_cookies)
* [Setting headers](#setting_headers)
* [Response filters](#response_filters)
* [Caching and compression](#caching_and_compression)

--------------------------------------------------------

The response class helps you build a HTTP response.

An instance of the response class is always available in all controller classes. It is also easily made available in [route closures](:base_url:/docs/:version:/routing-and-controllers:routing#basics).

--------------------------------------------------------

<a id="basics"></a>

### Basics

The ```body``` method allows you to set the response body. This is not normally needed as the framework automatically sets the response body to the return value of your controller/route action. It can however be useful if you want to alter the response body in an after filter.

	public function afterAction()
	{
		$this->response->body(strtoupper($this->response->getBody()));
	}

The ```getBody``` method returns the response body.

	$responseBody = $this->response->getBody();

The ```type``` method lets you define the response type as well as the response charset. The default response type is ```text/html``` and the default charset is ```UTF-8```. The default charset is defined in your ```app\config\application.php``` config file.

	$this->response->type('application/json');

	// You can also override the default charset

	$this->response->type('application/json', 'iso-8859-1');

The ```getType``` method returns the current response type.

	$responseType = $this->response->getType();

The ```charset``` method lets you set the response charset. The default charset is ```UTF-8```and it is defined in your ```app\config\application.php``` config file.

	$this->response->charset('iso-8859-1');

The ```getCharset``` returns the current response charset.

	$responseCharset = $this->response->getCharset();

The ```status``` method lets you set the response status.

	$this->response->status(404); // Sends the 404 "Not Found" header.

The ```getStatus``` method returns the current status code.

	$responseStatus = $this->response->getStatus();

--------------------------------------------------------

<a id="setting_cookies"></a>

### Setting cookies

The ```cookie``` method adds a cookie to the response.

	// Sets a cookie that expires when the browser closes

	$this->response->cookie('name', 'value');

	// Sets a cookie that expires after 1 hour

	$this->response->cookie('name', 'value', 3600);

	// You can also set the path, domain, secure and httponly options using the fourth parameter

	$this->response->cookie('name', 'value', 3600, ['path' => '/mydir', 'domain' => '.example.org']);

The ```signedCookie``` method has the same method signature as the ```cookie``` method. The difference is that the cookie will be signed using the application secret defined in the ```app/config/application.php``` config file.

	$this->response->signedCookie('name', 'value');

> The benefit of using signed cookies over regular cookies is that their values can not be tampered with on the client site.

The ```hasCookie``` method returns TRUE if the response has the specified cookie and FALSE if not.

	$hasCookie = $this->response->hasCookie('name');

The ```removeCookie``` method allows you to remove a cookie from the response.

	$this->response->removeCookie('name');

The ```deleteCookie``` method deletes a cookie from the browser. It can be used to delete both normal and signed cookies.

	$this->response->deleteCooke('name');

	// You can also set the path, domain, secure and httponly options using the second parameter

	$this->response->deleteCookie('name', ['path' => '/mydir', 'domain' => '.example.org']);

The ```getCookies``` method returns an array of all the cookies that have been set.

	$responseCookies = $this->response->getCookies();

The ```clearCookies``` method will clear all cookies that have been set.

	$this->response->clearCookies();

--------------------------------------------------------

<a id="setting_headers"></a>

### Setting headers

The ```header``` method adds a header to your response. The first parameter is the header field while the second is the header value.

	$this->response->header('access-control-allow-origin', '*');

The ```hasHeader``` method returns TRUE if the response has the specified header and FALSE if not.

	$hasHeader = $this->response->hasHeader('access-control-allow-origin'):

The ```removeHeader``` method allows you to remove a previously set header from the response.

	$this->response->removeHeader('access-control-allow-origin');

The ```getHeaders``` method will return an array of all the response headers that have been set.

	$responseHeaders = $this->response->getHeaders();

The ```clearHeaders``` method will clear all response headers that have been set

	$this->response->clearHeaders();

--------------------------------------------------------

<a id="response_filters"></a>

### Response filters

The ```filter``` methods allows you to add a response filter that the response body will be passed through before being sent to the browser. You can define multiple filters and they will be executed in the order that they were defined.

	$this->response->filter(function($responseBody)
	{
		return strtoupper($responseBody);
	});

The ```getFilters``` method will return an array containing all the defined response filters.

	$responseFilters = $this->response->getFilters();

The ```clearFilters``` method will clear all the defined response filters.

	$this->response->clearFilters();

--------------------------------------------------------

<a id="caching_and_compression"></a>

### Caching and compression

You can enable ```ETag``` caching using the ```cache``` method. Doing so can save bandwidth as the response body will only be sent if it has been modified since the last request.

	$this->response->cache();

The ```disableCaching``` method disables ```ETag``` caching if it has been enabled.

	$this->response->disableCaching();

The ```compress``` method enables output compression. This will save you bandwidth in exchange for a slight bump in CPU usage.

	$this->response->compress();

The ```disableCompression``` method disables output compression if it has been enabled.

	$this->response->disableCompression();
