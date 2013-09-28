# Views

--------------------------------------------------------

* [Basic views](#basic_views)
* [Template views](#template_views)
* [Custom renderers](#custom_renderers)
 
--------------------------------------------------------

Views is where the HTML, XML, etc ... of your application goes. A view can be an entire web page or just a part of a page like a header, menu or footer.

Using views allows you to separate the presentation and business logic of your application, permitting independent development, testing and maintenance of each.

All views must be located in the ```app/views``` directory. You can also create subdirectories to better organize your view files.

> Note that the default content type header sent by Mako is ```text/html```.

--------------------------------------------------------

<a id="basic_views"></a>

### Basic views

You create a view object by passing the name of the view file to the view constructor.

	// Loads the welcome.php view

	$view = new View('welcome');

	// Loads the bar.php view located in the foo directory

	$view = new View('foo.bar');

	// Loads the welcome.php view and assigns the "foo" variable with the value "bar"

	$view = new View('welcome', array('foo' => 'bar'));

You can also assign variables to a view by using the ```assign``` method of the view class. You can assign any kind of variable, even another view object. The assigned variable is only available in the view you assigned it to.

	// Assigned the variable $foo with the value "bar"

	$view->assign('foo', 'bar');

	// You can also assign variables using overloading

	$view->foo = 'bar';

You can also assign global view variables using the ```assignGlobal``` method.

	View::assignGlobal('user', $user);

The ```render``` method returns the rendered output of the view. You can pass a callback function or closure to filter the output.

	// Returns the rendered output

	$output = $view->render();

	// The view class implements the __toString method 
	// so you can just cast the view object to a string to render it

	$output = (string) $view;

	// Returns the rendered output after making the first character of each word in capitalized

	$output = $view->render(function($output)
	{
		return mb_convert_case(mb_strtolower($output), MB_CASE_TITLE);	
	});

--------------------------------------------------------

<a id="template_views"></a>

### Template views

Mako also includes a basic templating language that offers a simpler and less verbose syntax than plain PHP in addition to automatic escaping of all variables.

There is almost no overhead associated with using template views as they get compiled into regular PHP views and they won't be re-compiled until you update them.

> You must use the special ```.tpl.php``` extension on your views for them to get rendered using the template engine.

Printing an escaped variable is done using the following syntax:

	{{$foo}}

The preserve filter will escape output but not double-encode existing html entities.

	{{preserve:$foo}}

If you want to print an un-escaped variable then you'll have to use the following syntax:

	{{raw:$foo}}

Sometimes you'll want to set a default value for a variable that might be empty:

	{{$foo || 'Default value'}}

You can also use functions and methods in your templates:

	<a href="{{URL::to('about')}}">{{__('about')}}</a>

Conditional statements (```if```, ```elseif``` and ```else```) are also supported:

	{% if($foo === $bar) %}
		$foo equals $bar
	{% else %}
		$foo is not equal to $bar
	{% endif %}

Loops is something you'll often need. Templates support ```foreach```, ```for``` and ```while``` loops. You can skip an iteration using ```continue``` or break out of the loop using ```break```:

	<ul>
	{% foreach($articles as $article) %}
		<li>{{$article->title}}</li>
	{% endforeach %}
	</ul>

Another useful feature is template inheritance. This allows you to define a parent "wrapper" view that you can easily extend. Lets say you save the following template as ```parent.tpl.php```:

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<meta charset="{{MAKO_CHARSET}}">
			<title>{{block:title}}Default Title{{endblock}}</title>
		</head>
		<body>
			<h1>Header</h1>
			<ul>
			{{block:list}}{{endblock}}
			</ul>
			<hr>
			{{block:content}}{{endblock}}
			<hr>
			Footer
		</body>
	</html>

You can then create a child.tpl.php file that extends the parent template.

	{% extends:parent %}

	{% block:title %}My title{% endblock %}

	{% block:list %}
		<li>Item 1</li>
		<li>Item 2</li>
	{% endblock %}

	{% block:content %}
		This is the content.
	{% endblock %}

Rendering the child template will result in the following output:

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<meta charset="UTF-8">
			<title>My title</title>
		</head>
		<body>
			<h1>Header</h1>
			<ul>
				<li>Item 1</li>
				<li>Item 2</li>
			</ul>
			<hr>
			This is the content
			<hr>
			Footer
		</body>
	</html>

--------------------------------------------------------

<a id="custom_renderers"></a>

### Custom renderers

You can also use your own custom renderer if the plain and template renderes do not satisfy your needs. 

Your renderer has to implement the ```mako\view\renderer\RendererInterface``` interface and you'll have to register it using the ```registerRenderer``` method.

	// All views with a .twig.php extention will now be rendered using the custom Twig renderer

	View::registerRenderer('.twig', '\app\lib\view\renderers\Twig');
