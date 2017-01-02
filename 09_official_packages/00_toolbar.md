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

	composer require mako/toolbar

Next, add the ```mako\toolbar\ToolbarPackage``` package to your ```app/config/application.php``` config file.

Finally, you need to make sure that the toolbar gets rendered. The quickest way of getting it up and running is to add the following code to your ```app/bootstrap.php``` file:

	$container->get('response')->filter(function($body) use ($container)
	{
    	$toolbar = $container->get('toolbar');

    	return str_replace('</body>', $toolbar->render() . '</body>', $body);
	});

> You can also achieve the same result using custom [middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware).

The toolbar also comes with a "included files" panel that is disabled by default. To enable it add the following line of code:

	$toolbar->addPanel(new IncludedFilesPanel($container->get('view')));

--------------------------------------------------------

<a id="creating_custom_panels"></a>

### Creating custom panels

Custom panels must implement the `PanelInterface` class. You can extend the `Panel` class which already implements some of the required methods.

In the following example we'll create a simple dummy panel. First, lets start by creating the toolbar class.

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

Next, we'll need to create a view for our toolbar.

	<div class="mako-panel-header">
		<span class="mako-title">Hello, World!</span>
	</div>

	<div class="mako-panel-content">

		<p>
			Hello, World!
		</p>

	</div>

All we have to do now is to add our custom panel to the toolbar:

	$toolbar->addPanel(new HelloWorldPanel($container->get('view')));
