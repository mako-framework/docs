# Views

--------------------------------------------------------

* [Basics](#basics)
* [View renderers](#view_renderers)
	- [Plain PHP](#view_renderers:plain_php) 
	- [Templates](#view_renderers:templates)
* [Custom view renderers](#custom_view_renderers)
 
--------------------------------------------------------

Views is where the HTML of your application goes. A view can be an entire web page or just a small section of a page like a header, menu or footer.

Using views allows you to separate the presentation and business logic of your application, permitting independent development, testing and maintenance of each.

All views must be located in the ```app/views``` directory. You can also create subdirectories to better organize your view files.

--------------------------------------------------------

<a id="basics"></a>

### Basics

You create a view object by passing the name of the view file to the ```create``` method of the view factory.

	$view = $this->view->create('welcome');

You can also organize your views in subdirectories. The example loads the ```bar``` view located in the ```foo``` directory.

	$view = $this->view->create('foo.bar');

Assigning variables can be done using the optional second parameter.

	$view = $this->view->create('foo.bar', ['foo' => 'bar']);

You can also assign variables to a view object by using the ```assign``` method of the view class. You can assign any kind of variable, even another view object. The assigned variable is only available in the view you assigned it to.

	$view->assign('foo', 'bar');

You can also assign global view variables using the ```assign``` method on the view factory instance.

	$this->view->assign('user', $user);

The ```render``` method returns the rendered output of the view.

	$output = $view->render();

You can also render a view directly. This method also accepts a second parameter of variables as well.

	$rendered = $this->view->render('foo.bar');

--------------------------------------------------------

<a id="view_renderers"></a>

### View renderers

<a id="view_renderers:plain_php"></a>

#### Plain PHP

<a id="view_renderers:templates"></a>

As the name suggests, plain PHP views are just HTML (or whatever you're using to present your data) and PHP.

	<div>
		<p>Hello, <?= $name; ?>!</p>
	</div>

You also have access to a few handy methods that you should use to escape untrusted data in your output.

	<div>
		<p>Hello, <?= $this->escapeHTML($name, $__charset__); ?>!</p>
	</div>

| Method          | Desciption                                          |
|-----------------|-----------------------------------------------------|
| escapeHTML      | Escapes data for use in a HTML body context.        |
| escapeAttribute | Escapes data for use in a HTML attribute context.   |
| escapeCSS       | Escapes data for use in a CSS context.              |
| escapeJS        | Escapes data for use in a JavaScript context.       |
| escapeURL       | Escapes data for use in a URI or parameter context. |

> The ```escapeAttribute``` method is not enough to securely escape complex attributes such as ```href```, ```src```, ```style```, or any of the event handlers like ```onmouseover```, ```onmouseout``` etc. It is extremely important that event handler attributes should be escaped with the ```escapeJS``` filter.

#### Templates

Mako includes a simple templating language that offers a simpler and less verbose syntax than plain PHP in addition to automatic escaping of all variables.

There is almost no overhead associated with using template views as they get compiled into regular PHP views and they won't be re-compiled until you update them.

> You must use the special ```.tpl.php``` extension on your views for them to get rendered using the template engine.

Printing an escaped variable for use in a HTML content context is done using the following syntax:

	{{$foo}}

The ```preserve``` filter will escape output but not double-encode existing html entities.

	{{preserve:$foo}}

The ```attribute``` filter will escape output for use in a HTML attribute context.

	{{attribute:$foo}}

> The ```attribute``` filter is not enough to securely escape complex attributes such as ```href```, ```src```, ```style```, or any of the event handlers like ```onmouseover```, ```onmouseout``` etc. It is extremely important that event handler attributes should be escaped with the ```js``` filter.

The ```js``` filter will escape output for use in a JavaScript context.

	{{js:$foo}}

The ```css``` filter will escape output for use in a CSS context.

	{{css:$foo}}

The ```url``` filter will escape output for use in a URI parameter context.

	{{url:$foo}}

If you want to print an un-escaped variable then you can use the ```raw``` filter.

	{{raw:$foo}}

Sometimes you'll want to set a default value for a variable that might be empty. This can be done like this.

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

	$this->view->registerRenderer('.twig', function()
	{
		$renderer = new foo\bar\TwigRenderer;

		$renderer->setCachePath('/tmp');

		return $renderer;
	});

> All custom renderers must implement the ```mako\view\renderers\RendererInterface``` interface.