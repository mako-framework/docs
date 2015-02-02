# Controllers

--------------------------------------------------------

* [Basics](#basics)
* [Controller filters](#controller_filters)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The role of a controller is to respond to a HTTP request and construct a response. See the [routing docs](:base_url:/docs/:version:/routing-and-controllers:routing) on how to direct HTTP requests to a controller action.

--------------------------------------------------------

<a id="basics"></a>

### Basics

Here's a basic example controller that extends the base controller included with the framework.

	namespace app\controllers;

	use mako\http\routing\Controller;

	class Home extends Controller
	{
		public function welcome()
		{
			return 'Hello, world!';
		}
	}

Passing parameters to your controller actions is easy. Just define a route with the parameters you need and add the corresponding parameters to your method.

	$routes->get('/articles/{id}', 'Article::view');

> Note that it is important that that the method parameter has the same name as the route parameter.

	public function view($id)
	{
		return $id;
	}

--------------------------------------------------------

<a id="controller_filters"></a>

### Controller filters

All controllers have two special methods. The ```beforeFilter``` method which gets executed right before the controller action and the ```afterFilter``` method which gets executed right after the controller action.

The controller action and ```afterFilter``` methods will be skipped if the ```beforeFilter``` returns data.

	public function beforeFilter()
	{
		if($this->gatekeeper->isGuest())
		{
			return $this->response->redirect($this->urlBuilder->toRoute('user:login'));
		}
	}

> Note that a controller action and its after filter will **not** be executed if the before filter returns data.

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Controllers are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

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