# Response

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The response class is responsible for sending output to the browser.

An instance of the response object is always available in controllers and it can be accessed like this:

	$this->response

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```body``` method can be used to override the response body in the ```after``` method of your controller.

	public function after()
	{
		// Will prefix the response with "Hello, world!";
		
		$this->response->body('Hello, world!' . $this->response);
	}

The ```status``` method is a helper method that makes it easy to send HTTP status headers. The status method will always reply using the same protocol version as the request.

	$this->response->status(404); // Sends the 404 "Not Found" header.

The ```type``` method lets you set the content type of your response. The default charset of the application is used if the second parameter is left empty.

	$this->response->type('application/json');

The ```redirect``` method allows you to redirect a user to a different URL. The default status sent by the method is ```302 Found``` but you can override this by setting the status code in the second parameter.

	// Redirects to http://example.org/foo/bar

	$this->response->redirect('foo/bar');

	// Redirects to http://example.com with the "301 Moved Permanently" header

	$this->response->redirect('http://example.com', 301);

The ```filter``` method allows you to filter the output before sending it.

	$this->response->filter(function($body)
	{
		return str_ireplace('[mako:assets]', Assets::location(), $body);
	});