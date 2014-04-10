# Controllers

--------------------------------------------------------

* [Basics](#basics)
* [Controller filters](#controller_filters)

--------------------------------------------------------

The role of a controller is to respond to a HTTP request and construct a response. See the [routing docs](:base_url:/docs/:version:/routing-and-controllers:routing) on how to direct HTTP requests to a controller action.

--------------------------------------------------------

<a id="basics"></a>

### Basics

All controllers must extend the ```mako\http\routing\Controller``` base controller. Every controller has two protected properties by default, a ```request``` instance and a ```response``` instance.

	namespace app\controllers;

	class Home extends \mako\http\routing\Controller
	{
		public function welcome()
		{
			return 'Hello, world!';
		}
	}

Passing parameters to your controller actions is easy. Just define a route with the parameters you need and add the corresponding parameters to your method.

	// In your routes.php file

	$routes->get('/articles/{id}', 'app\controllers\Article::view');

	// In your Articles controller class

	public function view($id)
	{
		return $id;
	}

> If any of your parameters are optional then you'll have to give them a default value when you define the method.

Controller instances are resolved through the [inversion of control container](:base_url:/docs/:version:/getting-started:dependency-injection). This allows you to easily inject your dependecies and avoid tight coupling of your code.

Just make sure that the first two constructor parameters are for the ```Request``` and ```Response``` instances.

	namespace app\controllers;

	use \mako\http\Request;
	use \mako\http\Response;
	use \mako\view\ViewFactory;

	class Home extends \mako\http\routing\Controller
	{
		protected $viewFactory;

		public function __construct(Request $request, Response $response, ViewFactory $viewFactory)
		{
			parent::__construct($request, $response);

			$this->viewFactory = $viewFactory;
		}
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