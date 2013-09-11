# Controllers

--------------------------------------------------------

* [Bacic controllers](#basic_controllers)
* [Restful controllers](#restful_controllers)
* [Before and after filters](#before_and_after_filters)

--------------------------------------------------------

The task of a controller is to receive input and decide what to do with it (e.g, fetch an article from a model, assign the data to a view and then return the view).

There is a one-to-one relationship between a URL and the controller action that gets executed. Visiting ```http://example.org/foo/bar``` will normally execute the ```bar``` action of the ```foo``` controller. This behavior can be changed by using custom routes.

Controllers are instantiated by the Request class. The instance of the request object is accessed using the ```$this->request``` property. The second property that is always available in a controller is an instance of the Response class that can be accessed using ```$this->response```.

> All controllers must be located in the app/controllers directory and extend the mako\Controller class.

--------------------------------------------------------

<a id="basic_controllers"></a>

### Basic controllers

If you save the following code as example.php and go to ```http://example.org/example``` then you'll be greeted by ```Hello, world!```.

	<?php

	namespace app\controllers;

	class Example extends \mako\Controller
	{
		public function action_index()
		{
			return "Hello, world!";
		}

		public function action_date()
		{
			return date('l jS \of F Y h:i:s A');
		}
	}

Visiting ```http://example.org/example/date``` will display the current date. You might have noticed that both actions are prefixed with ```action_```. This lets you define actions with the same name as reserved keywords in PHP (e.g, 'list' would not work while 'action_list' would).

Passing arguments to a controller action is easy as you can se in the example below.

	<?php

	namespace app\controllers;

	class Arguments extends \mako\Controller
	{
		public function action_color1($color)
		{
			return 'your favorite color is ' . $color;
		}

		public function action_color2($color = 'green')
		{
			return 'your favorite color is ' . $color;
		}
	}

Visiting ```http://example.org/arguments/color1/blue``` will display ```my favorite color is blue```. Going to ```http://example.org/arguments/color1``` will display a ```404 error```. This is because the argument is required. The second action has an optional argument so visiting ```http://example.org/arguments/color2``` will not produce an error but instead tell you that your favorite color is green.

--------------------------------------------------------

<a id="restful_controllers"></a>

### Restful controllers

Rest controllers are useful when creating a RESTful web service. Rest controllers are basically the same as normal controllers except that you prefix the actions with the HTTP request method they are meant to respond to.

All you have to do to make your controller RESTful is to set the ```RESTFUL``` constant of you controller to ```TRUE```.

	<?php

	namespace app\controllers;

	class Rest extends \mako\Controller
	{
		const RESTFUL = true;

		public function get_index()
		{
			return "Hello, world!";
		}

		public function post_index()
		{
			return date('l jS \of F Y h:i:s A');
		}
	}

The ```get_index``` method responds to GET requests and the ```post_index``` method responds to POST requests made to the same URL.

--------------------------------------------------------

<a id="before_and_after_filters"></a>

### Before and after filters

The ```before``` method is always executed before the requested action.

The ```before``` method is always executed before the requested action.