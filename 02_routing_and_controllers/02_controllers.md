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
* [Deferred tasks ](#deferred_tasks)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The role of a controller is to respond to a HTTP request and construct a response. See the [routing docs](:base_url:/docs/:version:/routing-and-controllers:routing) on how to direct HTTP requests to a controller action.

--------------------------------------------------------

### <a id="basics" href="#basics">Basics</a>

#### <a id="basics:getting_started" href="#basics:getting_started">Getting started</a>

Here's a basic example controller that extends the base controller included with the framework.

```
<?php

namespace app\http\controllers;

use mako\http\routing\Controller;

class Home extends Controller
{
	public function welcome()
	{
		return 'Hello, world!';
	}
}
```

#### <a id="basics:parameters" href="#basics:parameters">Parameters</a>

Passing parameters to your controller actions is easy. Just define a route with the parameters you need and add the corresponding parameters to your method.

```
$routes->get('/articles/{id}', [Article::class, 'view']);
```

> Note that it is important that that the method parameter has the same name as the route parameter.

```
public function view($id)
{
	return $id;
}
```

--------------------------------------------------------

### <a id="controller_helpers" href="#controller_helpers">Controller helpers</a>

If you're extending the Mako base controller then you'll get a set of useful convenience methods for free.

#### <a id="controller_helpers:file_response" href="#controller_helpers:file_response">File response</a>

The `fileResponse` method returns a file response sender. The file download will be resumable, something that can be very useful when downloading large files.

```
return $this->fileResponse('/path/to/file.ext');
```

You can set a custom file name, mime type, content disposition and a closure to be executed after a completed download using a set of chainable methods.

| Method name      | Description                                                                                      |
|------------------|--------------------------------------------------------------------------------------------------|
| setName          | The file name sent to the client                                                                 |
| setDisposition   | Content-disposition (default is attachment)                                                      |
| setType          | The framework will try to detect the mime type for you but you can override it using this method |
| done             | Closure that will be executed when the download has been completed                               |

	return $this->fileResponse('/path/to/file.ext')->setName('foo.ext')->setType('text/plain');

> Note that any errors that happen in the closure will not be displayed as it happens after the output has been sent to the client. You'll have to check your logs for errors.

#### <a id="controller_helpers:redirect_response" href="#controller_helpers:redirect_response">Redirect response</a>

The `redirectResponse` method returns a redirect response sender.

```
return $this->redirectResponse('http://example.org');
```

The method also allows you to use a [route name](:base_url:/docs/:version:/routing-and-controllers:routing#reverse_routing) instead of an URL.

```
return $this->redirectResponse('articles.view', ['id' => 10]);
```

The default status code is set to `302` (Found) but you can override it by using the chainable `setStatus` method.

```
// You can set the status using the Status enum

return $this->redirectResponse('http://example.org')->setStatus(Status::FOUND);

// Or by passing a valid status code integer

return $this->redirectResponse('http://example.org')->setStatus(302);
```

It is also possible to use the following methods to set and get the status code.

| Method name       | Description                   |
|-------------------|-------------------------------|
| movedPermanently  | Sets the status code to `301` |
| found             | Sets the status code to `302` |
| seeOther          | Sets the status code to `303` |
| temporaryRedirect | Sets the status code to `307` |
| permanentRedirect | Sets the status code to `308` |
| getStatus         | Returns the status code       |

#### <a id="controller_helpers:stream_response" href="#controller_helpers:stream_response">Stream response</a>

The `streamResponse` method returns a stream response sender. They can be useful when sending large amounts data as the data will be flushed to the client in chunks, thus minimizing your application memory usage.

It also allows you to begin transmitting dynamically-generated content before knowing the total size of the content.

```
return $this->streamResponse(function ($stream) {
	$stream->flush('Hello, world!');

	sleep(2);

	$stream->flush('Hello, world!');
}, 'text/plain');
```

You can also set and get the content type and character set using the following methods.

| Method name | Description               |
|-------------|---------------------------|
| setType     | Sets the content type     |
| getType     | Returns the content type  |
| setCharset  | Sets the character set    |
| getCharset  | Returns the character set |

> Stream responses might not always work as expected as some webservers and reverse proxies will buffer the output before sending it in its entirety. When using Nginx you can send the `X-Accel-Buffering` header with the value `no` to disable output buffering for the response.

#### <a id="controller_helpers:json_response" href="#controller_helpers:json_response">JSON response</a>

The `jsonResponse` method returns a JSON response builder. It will convert the provided data to JSON and set the correct content type header.

```
return $this->jsonResponse([1, 2, 3]);
```

If you want your API endpoint to be able to serve JSONP as well then you'll have to chain the `asJsonpWith` method.

```
return $this->jsonResponse([1, 2, 3])->asJsonpWith('callback');
```

You can also set and get the character set and status code using the following methods.

| Method name | Description               |
|-------------|---------------------------|
| setCharset  | Sets the character set    |
| getCharset  | Returns the character set |
| setStatus   | Sets the status code      |
| getStatus   | Returns the status code   |

--------------------------------------------------------

### <a id="controller_events" href="#controller_events">Controller events</a>

All controllers can have two special methods. The `beforeAction` method which gets executed right before the controller action and the `afterAction` method which gets executed right after the controller action.

The controller action and `afterAction` methods will be skipped if the `beforeAction` returns data.

```
public function beforeAction()
{
	if ($this->gatekeeper->isGuest()) {
		return $this->redirectResponse('user:login');
	}
}
```

> Note that a controller action and its after action will **not** be executed if the before action returns data.

--------------------------------------------------------

### <a id="deferred_tasks" href="#deferred_tasks">Deferred tasks</a>

Sometimes, you'll want to postpone slow tasks, such as sending an email or processing data, until after the response has been sent to the client. This can be achieved by using deferred tasks.

Deferred tasks are still processed within the same request, so memory limits and total execution time must be taken into consideration. If your tasks take more than a couple of seconds, you’re likely better off sending them to an asynchronous work queue, allowing them to be processed independently of web requests.

To use deferred tasks, simply inject the `DeferredTasks` class and add your task using the `defer` method.

```
<?php

namespace app\http\controllers;

use mako\application\DeferredTasks;

class DeferredTaskExample
{
	public function example(DeferredTasks $tasks): string
	{
		$tasks->defer(static function (LoggerInterface $log): void {
			sleep(5);

			$log->info('Deferred log entry.');
		});

		return 'Hello, world!';
	}
}

```

In the example above, `Hello, world!` will be returned to the client immediately, and a new log entry will be created 5 seconds later.

Task handlers can be any type of callable and are executed using the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection), allowing you to inject any dependencies you need.

> Note that deferred tasks will only be processed after the response has been sent to the client if your application runs on a FastCGI server, such as PHP-FPM. When running on a non-FastCGI server, the response will hang until the tasks are completed.

--------------------------------------------------------

### <a id="dependency_injection" href="#dependency_injection">Dependency injection</a>

Controllers are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

```
<?php

namespace app\http\controllers;

use mako\http\routing\Controller;
use mako\view\ViewFactory;

class Articles extends Controller
{
	public function __construct(
		protected ViewFactory $view
	) {
	}
}
```

You can also inject your dependencies directly into a method since controller actions are executed by the `Container::call()` method.

```
public function view(ViewFactory $view, $id)
{
	return $view->render('article', ['id' => $id]);
}
```

Controllers that extends the framework base controller are also `container aware`. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).
