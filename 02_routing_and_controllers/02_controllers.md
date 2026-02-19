# Controllers

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting_started)
	- [Parameters](#basics:parameters)
* [Controller events](#controller_events)
* [Deferred tasks ](#deferred_tasks)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The role of a controller is to respond to an HTTP request and construct a response. See the [routing documentation](:base_url:/docs/:version:/routing-and-controllers:routing) to learn how to direct HTTP requests to a controller action.

--------------------------------------------------------

### <a id="basics" href="#basics">Basics</a>

#### <a id="basics:getting_started" href="#basics:getting_started">Getting started</a>

Here’s a basic example controller that extends the base controller included with the framework. Controllers do not need to extend the base controller; however, doing so gives you access to the useful `redirectResponse` method, which allows you to redirect to named routes or external URLs.

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

Passing parameters to your controller actions is simple. Define a route that includes the parameters you need, then add matching parameters to your action method.

```
$routes->get('/articles/{id}', [Article::class, 'view']);
```

> Note: It is important that that the method parameter has the same name as the route parameter.

```
public function view($id)
{
	return $id;
}
```

--------------------------------------------------------

### <a id="controller_events" href="#controller_events">Controller events</a>

All controllers can define two special methods: `beforeAction` and `afterAction`. The `beforeAction` method is executed immediately before the controller action, and the `afterAction` method is executed immediately after the controller action.

If `beforeAction` returns a value, the controller action and the `afterAction` method will be skipped.

```
public function beforeAction()
{
	if ($this->gatekeeper->isGuest()) {
		return $this->redirectResponse('user:login');
	}
}
```

> Note: A controller action and its after action will **not** be executed if the before action returns data.

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

> Note: Deferred tasks will only be processed after the response has been sent to the client if your application runs on a FastCGI server, such as PHP-FPM. When running on a non-FastCGI server, the response will hang until the tasks are completed.

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
