# Controllers

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting_started)
	- [Parameters](#basics:parameters)
* [Controller helpers](#controller_helpers)
	- [File response](#controller_helpers:file_response)
	- [Redirect response](#controller_helpers:redirect_response)
	- [Stream response](#controller_helpers:stream_response)
	- [JSON response](#controller_helpers:json_response)
* [Controller events](#controller_events)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The role of a controller is to respond to a HTTP request and construct a response. See the [routing docs](:base_url:/docs/:version:/routing-and-controllers:routing) on how to direct HTTP requests to a controller action.

--------------------------------------------------------

<a id="basics"></a>

### Basics

<a id="basics:getting_started"></a>

#### Getting started

Here's a basic example controller that extends the base controller included with the framework.

	<?php

	namespace app\controllers;

	use mako\http\routing\Controller;

	class Home extends Controller
	{
		public function welcome()
		{
			return 'Hello, world!';
		}
	}

<a id="basics:parameters"></a>

#### Parameters

Passing parameters to your controller actions is easy. Just define a route with the parameters you need and add the corresponding parameters to your method.

	$routes->get('/articles/{id}', 'Article::view');

> Note that it is important that that the method parameter has the same name as the route parameter.

	public function view($id)
	{
		return $id;
	}

--------------------------------------------------------

<a id="controller_helpers"></a>

### Controller helpers

If you're extending the Mako base controller then you'll get a set of useful convenience methods for free.

<a id="controller_helpers:file_response"></a>

#### File response

The ```fileResponse``` method returns a file response sender. The file download will be resumable, something that can be very useful when downloading large files.

	return $this->fileResponse('/path/to/file.ext');

You can set a custom file name, mime type, content disposition and a closure to be executed after a completed download using a set of chainable methods.

| Method name   | Description                                                                                      |
|---------------|--------------------------------------------------------------------------------------------------|
| name          | The file name sent to the client                                                                 |
| disposition   | Content-disposition (default is attachment)                                                      |
| type          | The framework will try to detect the mime type for you but you can override it using this method |
| done          | Closure that will be executed when the download has been completed                               |

	return $this->fileResponse('/path/to/file.ext')->name('foo.ext')->type('text/plain');

> Note that any errors that happen in the closure will not be displayed as it happens after the output has been sent to the client. You'll have to check your logs for errors.

<a id="controller_helpers:redirect_response"></a>

#### Redirect response

The ```redirectResponse``` method returns a redirect response sender.

	return $this->redirectResponse('http://example.org');

You can also set the status header (default is 302) by chaining the status method.

	return $this->redirectResponse('http://example.org')->status(301);

The ```redirectResponse``` method also allows you to use a [route name](:base_url:/docs/:version:/routing-and-controllers:routing#reverse_routing) instead of an URL.

	return $this->redirectResponse('articles.view', ['id' => 10]);

<a id="controller_helpers:stream_response"></a>

#### Stream response

The ```streamResponse``` method returns a stream response sender. They can be useful when sending large amounts data as the data will be flushed to the client in chunks, thus minimizing your application memory usage.

It also allows you to begin transmitting dynamically-generated content before knowing the total size of the content.

	$this->response->type('text/plain');

	return $this->streamResponse(function($stream)
	{
		$stream->flush('Hello, world!');

		sleep(2);

		$stream->flush('Hello, world!');
	});

> Stream responses might not always work as expected as some webservers and reverse proxies will buffer the output before sending it.

<a id="controller_helpers:json_response"></a>

#### JSON response

The ```jsonResponse``` method returns a JSON response builder. It will convert the provided data to JSON and set the correct content type header.

	return $this->jsonResponse([1, 2, 3]);

If you want your API endpoint to be able to serve JSONP as well then you'll have to chain the ```asJsonpWith``` method.

	return $this->jsonResponse([1, 2, 3])->asJsonpWith('callback');

You can also set the HTTP status code using the ```status``` method.

	return $this->jsonResponse(['message' => 'Page not found'])->status(404);

--------------------------------------------------------

<a id="controller_events"></a>

### Controller events

All controllers have two special methods. The ```beforeAction``` method which gets executed right before the controller action and the ```afterAction``` method which gets executed right after the controller action.

The controller action and ```afterAction``` methods will be skipped if the ```beforeAction``` returns data.

	public function beforeAction()
	{
		if($this->gatekeeper->isGuest())
		{
			return $this->redirectResponse('user:login');
		}
	}

> Note that a controller action and its after action will **not** be executed if the before action returns data.

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Controllers are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

	<?php

	namespace app\controllers;

	use mako\http\routing\Controller;
	use mako\view\ViewFactory;

	class Articles extends Controller
	{
		protected $view;

		public function __construct(ViewFactory $view)
		{
			$this->view = $view;
		}
	}

You can also inject your dependencies directly into a method since controller actions are executed by the ```Container::call()``` method.

	public function view(ViewFactory $view, $id)
	{
		return $view->render('article', ['id' => $id]);
	}

Controllers that extends the framework base controller are also ```container aware```. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).
