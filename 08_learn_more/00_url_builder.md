# URL builder

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The URL builder helps you build URLs for your Mako application.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```to``` method will return a Mako URL to the chosen path.

	// Will print http://example.org/foo/bar

	$url = $urlBuilder->to('foo/bar');

	// Will print http://example.org/foo/bar?key=value&key2=value2

	$url = $urlBuilder->to('foo/bar', ['key1' => 'value1', 'key2' => 'value2']);

The ```toRoute``` returns the URL of a [named route](:base_url:/docs/:version:/routing-and-controllers:routing#reverse_routing).

	$url = $urlBuilder->toRoute('home');

The ```base``` method will return the base URL of your application.

	$url = $urlBuilder->base();

The ```current``` method will return the current URL of the main request.

	$url = $urlBuilder->current();

The ```matches``` method returns TRUE if the current URL path matches the pattern and false if not.

	// Basic URL matching

	$matching = $urlBuilder->matches('/news');

	// You can also use regular expressions

	$matching = $urlBuilder->matches('/news/article/([0-9]+)');
