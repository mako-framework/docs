# REST client

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a basic REST that client makes it easy to interact with [RESTful web services](http://en.wikipedia.org/wiki/Representational_state_transfer#RESTful_web_services).

--------------------------------------------------------

<a id="usage"></a>

### Usage

First you'll need to create a Rest object. Use the optional options parameter to set [CURL options](http://no2.php.net/manual/en/function.curl-setopt.php).

	$rest = new Rest('http://example.org/api');

	$rest = new Rest('http://example.org/api', array(CURLOPT_CONNECTTIMEOUT => 1));

If you need to provide authentication then you can use the ```authenticate``` method.

	$rest->authenticate('username', 'password');

The ```get``` method performs a GET request and returns the response.

	$response = $rest->get();

The ```head``` method performs a HEAD request and returns the response headers in a nicely formatted array.

	$headers = $rest->head();

The ```post``` method performs a POST request and returns the response.

	$movie = array
	(
		'title'    => 'Thor',
		'category' => 'Action',
	);

	$response = $rest->post($movie);

The ```put``` method performs a PUT request and returns the response.

	$movie = array
	(
		'title'    => 'Thor',
		'category' => 'Action',
	);

	$response = $rest->put(json_encode($movie));

The ```delete``` method performs a DELETE request and returns the response.

	$response = $rest->delete();

The ```info``` method returns info about the last cURL request.

	$rest = new Rest('http://example.org/api');

	$response = $rest->get();

	if($rest->info('http_code') != 200)
	{
		// something went wrong
	}
	else
	{
		// everything is ok
	}

You can also set the ```$info``` parameter offered by the request methods to save a few lines of code.

	$response = Rest::factory('http://example.org')->get($info);

	if($info['http_code'] != 200)
	{
		// something went wrong
	}
	else
	{
		// everything is ok
	}