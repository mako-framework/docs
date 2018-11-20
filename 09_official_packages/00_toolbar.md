# Toolbar

--------------------------------------------------------

* [Installation](#installation)
* [Creating custom panels](#creating_custom_panels)

--------------------------------------------------------

The Mako Toolbar package is a useful developer tool that allows you to take a look at the request, response, configuration, session data, database queries, execution time and memory usage of your application.

--------------------------------------------------------

<a id="installation"></a>

### Installation

Install the package using the following composer command:

```
composer require mako/toolbar
```
{.language-none}

Next, add the `mako\toolbar\ToolbarPackage` package to your `app/config/application.php` config file.

Finally, you need to make sure that the toolbar gets rendered. The quickest way of getting it up and running is to use the included [middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware).

```
$dispatcher->registerMiddleware('toolbar', ToolbarMiddleware::class);
```

You should make sure that the middleware gets executed first to ensure that the toolbar is able to collect all the information about your application.

```
$dispatcher->setMiddlewarePriority(['toolbar' => 1]);
```

You can now add the middleware to the routes of your choice or make it global if you want to apply it to all your routes.

```
$dispatcher->setMiddlewareAsGlobal(['toolbar']);
```

> The middleware will only append the toolbar to responses with a content type of `text/html` and a body that includes a set of `<body></body>` tags.

--------------------------------------------------------

<a id="creating_custom_panels"></a>

### Creating custom panels

Custom panels must implement the `PanelInterface` class. You can extend the `Panel` class which already implements some of the required methods.

In the following example we'll create a simple dummy panel. First, lets start by creating the toolbar class.

```
<?php

namespace app\toolbar\panels;

use mako\toolbar\panels\Panel;
use mako\toolbar\panels\PanelInterface;

class HelloWorldPanel extends Panel implements PanelInterface
{

	public function getTabLabel(): string
	{
		return 'Hello, World!';
	}

	public function render(): string
	{
		return $this->view->render('toolbar.panels.hello_world');
	}
}
```

Next, we'll need to create a view for our toolbar.

```
<div class="mako-panel-header">
	<span class="mako-title">Hello, World!</span>
</div>

<div class="mako-panel-content">

	<p>
		Hello, World!
	</p>

</div>
```

All we have to do now is to add our custom panel to the toolbar:

```
$toolbar->addPanel(new HelloWorldPanel($container->get(ViewFactory::class)));
```
