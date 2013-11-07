# Asset manager

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The asset manager makes it easy to organize, group and display assets (CSS and JavaScript) in your views.

> The asset manager will return HTML5 tags but you can make it return XHTML compatible tags by defining the ```MAKO_XHTML``` constant.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```location``` method will return the location of your assets as defined in app/config/application.php.

	echo Assets::location();

The ```add``` method will add an asset to the manager. It can either be a URL or a path. If a path is used then the asset manager will automatically prefix your path with the path/url to the assets directory.

	// Add external JavaScript

	Assets::add('jquery', 'https://ajax.googleapis.com/ajax/libs/jquery/1.7.1/jquery.min.js');
	Assets::add('jquery-ui', 'https://ajax.googleapis.com/ajax/libs/jqueryui/1.8.16/jquery-ui.min.js');

	// "/css/*.css" will now be prefixed with the path/url to your asset directory

	Assets::add('web-style', '/css/web.css');
	Assets::add('print-style', '/css/print.css', array('media' => 'print'));

The ```css``` method will return one or all CSS assets.

	// Will print the link tag for the web-style

	<?php echo Assets::css('web-style'); ?>

	// Will print the link tags for all styles

	<?php echo Assets::css(); ?>

The ```js``` method will return one or all JavaScript assets.

	// Will print the script tag for jquery

	<?php echo Assets::js('jquery'); ?>

	// Will print the script tags for all scripts

	<?php echo Assets::js(); ?>

The ```all``` method will return all assets that have been defined.

	<?php echo Assets::all(); ?>

Sometimes you might want to group your assets. This is easy using the ```group``` method.

	// Will add "/js/myscript.js" to the header group

	Assets::group('header')->add('myscript', '/js/myscript.js');

	// Will add "/js/myotherscript.js" to the footer group

	Assets::group('footer')->add('myotherscript', '/js/myotherscript.js');

Fetching your grouped assets is just as easy.

	<?php echo Assets::group('header')->all(); ?>