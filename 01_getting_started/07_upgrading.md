# Upgrading

--------------------------------------------------------

* [Application](#application)
	- [Structure](#application:structure) 
	- [Configuration](#application:configuration) 
	- [Route filters](#application:route_filters)
* [Packages](#packages)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```4.1.x``` to ```4.2.x```.

> Remember to clear your cached views (located in ```app/storage/cache/views```) if you're using templates.

--------------------------------------------------------

<a id="application"></a>

### Application

<a id="application:structure"></a>

#### Structure

The first thing you have to do is creating an ```app/routing``` directory and move your ```app/routes.php``` file into it. Then you'll have to create a ```filters.php``` file in your newly created ```app/routing``` directory.

<a id="application:configuration"></a>

#### Configuration

Next you'll have to update your ```app/config/application.php``` configuration file. Remove the ```locale``` setting and update the [default_language](https://github.com/mako-framework/app/blob/master/app/config/application.php#L54) and [languages](https://github.com/mako-framework/app/blob/master/app/config/application.php#L65) settings to the new format. And finally add the new [packages](https://github.com/mako-framework/app/blob/master/app/config/application.php#L149) configuration option.

The ```app/config/gatekeeper.php``` configuration file has also been updated. All you have to here is adding the [identifier](https://github.com/mako-framework/app/blob/master/app/config/gatekeeper.php#L13) option. It'll let you choose between identifying users using their email address or username.

<a id="application:route_filters"></a>

#### Route filters

Route filters are no longer defined in the ```routes.php``` file but it a separate ```filters.php``` file. The [syntax](:base_url:/docs/:version:/routing-and-controllers:routing#route_filters) remains the same so migrating your filters to the 4.2 way of doing things should only take a minute or two.

--------------------------------------------------------

<a id="packages"></a>

### Packages

The biggest change in Mako 4.2 is how packages work. Take a look at the [package documentation](:base_url:/docs/:version:/packages:packages) to see what changes you will need to make to your packages.

You can also take a look at the [Mako Toolbar](https://github.com/mako-framework/toolbar) package to get an idea of how 4.2 packages are structured.