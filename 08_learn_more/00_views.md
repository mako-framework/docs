# Views

--------------------------------------------------------

* [Basics](#basics)
* [View renderers](#view_renderers)
	- [Plain PHP](#view_renderers:plain_php)
	- [Templates](#view_renderers:templates)
		- [Printing](#view_renderers:templates:printing)
		- [Conditional statements](#view_renderers:templates:conditional_statements)
		- [Loops](#view_renderers:templates:loops)
		- [Capturing blocks](#view_renderers:templates:capturing_blocks)
		- [Nospace blocks](#view_renderers:templates:nospace_blocks)
		- [Verbatim blocks](#view_renderers:templates:verbatim_blocks)
		- [Including views](#view_renderers:templates:including_views)
		- [Template inheritance](#view_renderers:templates:template_inheritance)
* [Custom view renderers](#custom_view_renderers)

--------------------------------------------------------

Views is where the HTML of your application goes. A view can be an entire web page or just a small section of a page like a header, menu or footer.

Using views allows you to separate the presentation and business logic of your application, permitting independent development, testing and maintenance of each.

All views must be located in the ```app/resources/views``` directory. You can of course create subdirectories to better organize your view files.

--------------------------------------------------------

<a id="basics"></a>

### Basics

Creating a view object is done by passing the name of the view file to the ```create``` method of the view factory.

	$view = $this->view->create('welcome');

If you're organizing your views in subdirectories then you'll have to separate the directory and template names by a dot.  The following example loads the ```bar``` view located in the ```foo``` directory.

	$view = $this->view->create('foo.bar');

Assigning variables can be done using the optional second parameter.

	$view = $this->view->create('foo.bar', ['foo' => 'bar']);

You can also assign variables to a view object by using the ```assign``` method of the view class. You can assign any kind of variable, even another view object. The assigned variable is only available in the view you assigned it to and in views included within it.

	$view->assign('foo', 'bar');

It is also possible to auto assign variables to views using the view factory ```autoAssign``` method. The first parameter is the name of the view (or an array of view names) and the second parameter is a callable that must return an associative array where the keys are the variable names and the values are the variable values.

	$this->view->autoAssign('profile', function()
	{
		['name' => 'Foobar'];
	});

You can also assign global view variables that will be available in all views using the ```assign``` method on the view factory instance.

	$this->view->assign('user', $user);

> You can also use ```*``` as a template name to automatically assign a variable to all views.

The ```render``` method returns the rendered output of the view and it also accepts the same optional second parameter as the create method.

	$output = $view->render();

You can also render a view directly from the view factory, and as expected it accepts the same optional second parameter as the render method of the view object.

	$rendered = $this->view->render('foo.bar');

--------------------------------------------------------

<a id="view_renderers"></a>

### View renderers

<a id="view_renderers"></a>

#### Plain PHP

<a id="view_renderers:plain_php"></a>

As the name suggests, plain PHP views are just HTML (or whatever you're using to present your data) and PHP.

	<div>
		<p>Hello, <?= $name; ?>!</p>
	</div>

You also have access to a few handy methods that you should use to escape untrusted data in your output.

	<div>
		<p>Hello, <?= $this->escapeHTML($name, $__charset__); ?>!</p>
	</div>

| Method          | Description                                         |
|-----------------|-----------------------------------------------------|
| escapeHTML      | Escapes data for use in a HTML body context.        |
| escapeAttribute | Escapes data for use in a HTML attribute context.   |
| escapeCSS       | Escapes data for use in a CSS context.              |
| escapeJS        | Escapes data for use in a JavaScript context.       |
| escapeURL       | Escapes data for use in a URI or parameter context. |

> The ```escapeAttribute``` method is not enough to securely escape complex attributes such as ```href```, ```src```, ```style```, or any of the event handlers like ```onmouseover```, ```onmouseout``` etc. It is extremely important that event handler attributes should be escaped with the ```escapeJS``` filter.
{.warning}

#### Templates

<a id="view_renderers:templates"></a>

Mako includes a simple templating language that offers a simpler and less verbose syntax than plain PHP in addition to automatic escaping of all variables.

There is almost no overhead associated with using template views as they get compiled into regular PHP views and they won't be re-compiled until you update them.

> Your views must use the custom ```.tpl.php``` extension for them to get rendered by the template renderer.

##### Printing

<a id="view_renderers:templates:printing"></a>

Printing an escaped variable for use in a HTML content context is done using the following syntax:

	{{$foo}}

The ```preserve``` filter will still escape output but it will prevent double-encoding of existing html entities.

	{{preserve:$foo}}

The ```attribute``` filter will escape output for use in a HTML attribute context.

	{{attribute:$foo}}

> The ```attribute``` filter is not enough to securely escape complex attributes such as ```href```, ```src```, ```style```, or any of the event handlers like ```onmouseover```, ```onmouseout``` etc. It is extremely important that event handler attributes should be escaped with the ```js``` filter.
{.warning}

The ```js``` filter will escape output for use in a JavaScript context.

	{{js:$foo}}

The ```css``` filter will escape output for use in a CSS context.

	{{css:$foo}}

The ```url``` filter will escape output for use in a URI parameter context.

	{{url:$foo}}

If you want to print an un-escaped variable then you can use the ```raw``` filter.

	{{raw:$foo}}

Sometimes you'll want to print a default value if your variable is empty. This can easily be achieved using the following syntax.

	{{$foo || 'Default value'}}

##### Conditional statements

<a id="view_renderers:templates:conditional_statements"></a>

Conditional statements (```if```, ```elseif``` and ```else```) are also supported.

	{% if($foo === $bar) %}
		$foo equals $bar
	{% else %}
		$foo is not equal to $bar
	{% endif %}

##### Loops

<a id="view_renderers:templates:loops"></a>

Loops is something you'll often need when displaying data. Templates support ```foreach```, ```for``` and ```while``` loops. You can skip an iteration using ```continue``` or break out of the loop using ```break```.

	<ul>
	{% foreach($articles as $article) %}
		<li>{{$article->title}}</li>
	{% endforeach %}
	</ul>

##### Capturing blocks

<a id="view_renderers:templates:capturing_blocks"></a>

It is possible to render and capture parts of a template for later use by using a `capture` block.

	{% capture:captured %}
		<p>Hello, world!</p>
	{% endcapture %}

In the example above, you'll be able to access the captured block using a variable named `$captured` anywhere after the capture closing tag.

##### Nospace blocks

<a id="view_renderers:templates:nospace_blocks"></a>

Sometimes you'll have to write markup without whitespace between tags as a workaround for browser rendering quirks. This can quickly lead to messy and hard to read templates. This is where the `nospace` block comes in handy.

	{% nospace %}
		<div>
			<p>Hello, world!</p>
		</div>
	{% endnospace %}

All whitespace between the tags will be removed at compile time and it will not affect rendering times at all.

If you want to render a dynamic block where all whitespace between tags has been removed then you'll want to add the `buffered` flag. This will delay the whitespace removal until the content is being rendered.

	{% nospace:buffered %}
		<ul>
			{% foreach(range(1, 10) as $number) %}
				<li>{{$number}}</li>
			{% endforeach %}
		</ul>
	{% endnospace %}

> Nospace blocks will remove whitespace between _all_ tags. If you need a space character between blocks then you can use the `&ensp;` entity.

##### Verbatim blocks

<a id="view_renderers:templates:verbatim_blocks"></a>

If you want the template compiler to ignore a section of your template then you can use the `verbatim` blocks.

	{% verbatim %}
		This {{$will}} not be {{$parsed}}.
	{% endverbatim %}

##### Including views

<a id="view_renderers:templates:including_view"></a>

You can easily include a partial template in a view.

	{{view:'partials.footer'}}

Included views will automatically inherit all the variables available in the parent view but you can override them or add new ones.

	{{view:'partials.footer', ['foo' => 'bar']}}

##### Template inheritance

<a id="view_renderers:templates:template_inheritance"></a>

Another useful feature is template inheritance. This allows you to define a parent wrapper view that you can easily extend. Let's say you save the template below as ```parent.tpl.php```.

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

You can then create a ```child.tpl.php``` template that extends the parent template.

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

Rendering the child template will result in the HTML document displayed below.

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

Mako also makes it possible to use custom view renderers such as [Twig](http://twig.sensiolabs.org/) or [Smarty](http://www.smarty.net/).

Registering a custom renderer is done using the ```extend``` method. The first parameter is the file extension you want to associate with your custom renderer and the second parameter is the class name of your renderer class.

The renderer will be instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so all dependencies will automatically be injected.

	$this->view->extend('.twig', TwigRenderer::class);

You can also use a closure if your renderer requires parameters that the container is unable to resolve.

	$this->view->extend('.twig', function()
	{
		$renderer = new foo\bar\TwigRenderer;

		$renderer->setCachePath('/tmp');

		return $renderer;
	});

> All custom renderers must implement the ```mako\view\renderers\RendererInterface``` interface.
