# Views

--------------------------------------------------------

* [Basics](#basics)
* [Template views](#template_views)
* [Custom view renderers](#custom_view_renderers)
 
--------------------------------------------------------

Views is where the HTML of your application goes. A view can be an entire web page or just a small section of a page like a header, menu or footer.

Using views allows you to separate the presentation and business logic of your application, permitting independent development, testing and maintenance of each.

All views must be located in the ```app/views``` directory. You can also create subdirectories to better organize your view files.

--------------------------------------------------------

<a id="basics"></a>

### Basics

You create a view object by passing the name of the view file to the ```create``` method of the view factory.

	// Loads the welcome.php view

	$view = $this->view->create('welcome');

	// Loads the bar.php view located in the foo directory

	$view = $this->view->create('foo.bar');

	// Loads the welcome.php view and assigns the "foo" variable with the value "bar"

	$view = $this->view->create('foo.bar', ['foo' => 'bar']);

You can also assign variables to a view by using the ```assign``` method of the view class. You can assign any kind of variable, even another view object. The assigned variable is only available in the view you assigned it to.

	$view->assign('foo', 'bar');

You can also assign global view variables using the ```assign``` method on the view factory instance.

	$this->view->assign('user', $user);

The ```render``` method returns the rendered output of the view.

	$output = $view->render();

--------------------------------------------------------

<a id="template_views"></a>

### Template views

Mako includes a simple templating language that offers a simpler and less verbose syntax than plain PHP in addition to automatic escaping of all variables.

There is almost no overhead associated with using template views as they get compiled into regular PHP views and they won't be re-compiled until you update them.

> You must use the special ```.tpl.php``` extension on your views for them to get rendered using the template engine.

Printing an escaped variable is done using the following syntax:

	{{$foo}}

The preserve filter will escape output but not double-encode existing html entities.

	{{preserve:$foo}}

If you want to print an un-escaped variable then you'll have to use the following syntax:

	{{raw:$foo}}

Sometimes you'll want to set a default value for a variable that might be empty. This can be done like this:

	{{$foo || 'Default value'}}

You can also use functions and methods in your templates:

	<a href="{{$urlbuilder->to('about')}}">{{$i18n->get('about')}}</a>

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

You can also include a view inside another view like this:

	{{view:'partials.footer'}}

Another useful feature is template inheritance. This allows you to define a parent wrapper view that you can easily extend. Lets say you save the following template as ```parent.tpl.php```:

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<meta charset="{{$__charset__}}">
			<title>{{block:title}}My Site{{endblock}}</title>
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

You can then create a ```child.tpl.php``` file that extends the parent template.

	{% extends:'parent' %}

	{% block:title %}My Page - __PARENT__{% endblock %}

	{% block:list %}
		<li>Item 1</li>
		<li>Item 2</li>
	{% endblock %}

	{% block:content %}
		This is the content.
	{% endblock %}

> The ```__PARENT__``` string will be replaced by the contents of the parent block.

Rendering the child template will result in the following output:

	<!DOCTYPE html>
	<html lang="en">
		<head>
			<meta charset="UTF-8">
			<title>My Page - My Site</title>
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

<a id="custom_view_renderers"></a>

### Custom view renderers

Mako also makes it easy to use custom view renderers such as [Twig](http://twig.sensiolabs.org/) or [Smarty](http://www.smarty.net/). 

Registering a custom renderer is done using the ```registerRenderer``` method. The first parameter is the file extention you want to associate with your custom renderer. The second parameter is the namespaced class name of your renderer class.

	$this->view->registerRenderer('.twig', 'foo\bar\TwigRenderer');

You can also use a closure when registering a custom renderer.

	$this->view->registerRenderer('.twig', function($view, $parameters)
	{
		// Return an implementation of mako\view\renderers\RendererInterface
	});

> All custom renderers must implement the ```mako\view\renderers\RendererInterface``` interface.