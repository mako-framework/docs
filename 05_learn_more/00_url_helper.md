# URL helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The URL helper contains methods for generating Mako URLs.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```to``` method will return a Mako URL to the chosen route.

	// Will print http://example.org/foo/bar

	echo URL::to('foo/bar');

	// Will print http://example.org/foo/bar?key=value&key2=value2

	echo URL::to('foo/bar', array('key1' => 'value1', 'key2' => 'value2'));

The ```current``` method will return the current URL of the main request.

	// Visiting http://example.org/foo/bar will print http://example.org/foo/bar

	echo URL::current();

	// Visiting http://example.org/foo/bar/baz will print http://example.org/foo/bar/baz

	echo URL::current();

The ```matches``` method returns TRUE if the current URL matches the pattern and false if not.

	// Basic URL matching

	if(URL::matches('news'))
	{

	}

	// You can also use regular expressions to match URLs

	if(URL::matches('news/article/([0-9]+)'))
	{

	}