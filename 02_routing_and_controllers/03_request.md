# Request

--------------------------------------------------------

* [Request data](#request_data)
	- [Query string, post data and parsed body](#request_data:query_string_post_data_and_parsed_body)
	- [Raw request body](#request_data:raw_request_body)
* [Cookies](#cookies)
* [Headers](#headers)
* [Files](#files)
* [Server information](#server_information)
* [Request information](#request_information)

--------------------------------------------------------

The request class provides an object oriented interface to global variables such as `$_GET`, `$_POST`, `$_FILES`, `$_COOKIE` and `$_SERVER` as well as easy access to other useful request information.

--------------------------------------------------------

<a id="request_data"></a>

### Accessing data

<a id="request_data:query_string_post_data_and_parsed_body"></a>

#### Query string, post data and parsed body

The `getQuery` method returns a parameter collection containing query string data.

	$queryString = $this->request->getQuery();

The `get` method of the parameter collection returns the value of a parameter and `null` if it doesn't exist.

	$value = $queryString->get('foo');

	// You can also specify a custom default return value

	$value = $queryString->get('foo', false);

The `getPost` returns a parameter collection containing post data.

	// Get all post data

	$post = $this->request->getPost();

The `getBody` method returns a parameter collection containing data from a parsed request body. The method currently supports `application/x-www-form-urlencoded` and `application/json` request bodies.

	$body = $this->request->getBody();

The `getData` method returns a parameter collection containing the data corresponding to the request method (query string for `GET` requests, post data for `POST` requests and the parsed body for any other type of request).

	$data = $this->request->getData();

The the parameter collections also include the following methods in addition to the ones shown in the examples above.

| Method name | Description                                                                 |
|-------------|-----------------------------------------------------------------------------|
| all         | Returns an array containing all the parameters                              |
| add         | Adds a parameter                                                            |
| has         | Returns `true` if the collection has the parameter and `false` if not       |
| remove      | Removes a parameter                                                         |
| whitelisted | Returns a whitelisted array of parameters                                   |
| blacklisted | Returns an array of parameters where the blacklisted keys have been removed |

<a id="request_data:raw_request_body"></a>

#### Raw request body

The `getRawBody()` method returns the raw request body as a string.

	$body = $this->request->getRawBody();

The `getRawBodyAsStream` method returns the raw request body as a stream. This can be useful if you want to minimize the memory footprint of your application while handling large request bodies.

	$body = $this->request->getRawBodyAsStream();

	$storageStream = fopen($this->app->getPath() . '/storage/uploads/' . $fileName, 'w');

	$filesize = stream_copy_to_stream($body, $storageStream);

--------------------------------------------------------

<a id="reading_cookies"></a>

### Reading cookies

The `getCookies` returns a cookie collection that allows to you access both signed and unsigned cookies.

	$cookies = $this->request->getCookies();

The `get` method of the cookie collection returns the value of a cookie and `null` if it doesn't exist.

	$value = $cookies->get('foobar');

	// You can also define a custom default return value

	$value = $cookies->get('foobar', false);

> If you want to read a signed cookie then you'll have to use the `getSigned` method. The benefit of using signed cookies is that they can't be tampered with on the client side.

The cookie collection also includes the following methods in addition to the ones shown in the examples above.

| Method name | Description                                            |
|-------------|--------------------------------------------------------|
| all         | Returns an array containing all the cookies            |
| add         | Adds a cookie                                          |
| addSigned   | Adds a signed cookie                                   |
| has         | Returns `true` if the cookie exists and `false` if not |
| remove      | Removes a cookie                                       |

--------------------------------------------------------

<a id="headers"></a>

### Headers

The `getHeaders` method returns a header collection.

	$headers = $this->request->getHeaders();

The `get` method of the header collection returns the value of the header an `null` it it isn't set.

	$value = $headers->get('content-type');

	// You can also define a custom default return value

	$value = $headers->get('content-type', 'text/plain');

The `acceptableContentTypes` method of the header collection returns an array of acceptable content types in descending order of preference.

	$contentTypes = $headers->acceptableContentTypes();

The `acceptableLanguages` method of the header collection returns an array of acceptable languages in descending order of preference.

	$languages = $headers->acceptableLanguages();

The `acceptableCharsets` method of the header collection returns an array of acceptable character sets in descending order of preference.

	$charsets = $headers->acceptableCharsets();

The `acceptableEncodings` method of the header collection returns an array of acceptable encodings in descending order of preference.

	$charsets = $headers->acceptableEncodings();

The header collection also includes the following methods in addition to the ones shown in the examples above.

| Method name | Description                                            |
|-------------|--------------------------------------------------------|
| all         | Returns an array containing all the headers            |
| add         | Adds a header                                          |
| has         | Returns `true` if the header exists and `false` if not |
| remove      | Removes a header                                       |

--------------------------------------------------------

<a id="files"></a>

### Files

The `getFiles` method returns a file collection that extends the parameter collection.

	$files = $this->request->getFiles();

The `get` method of the file collection returns a `UploadedFile` instance or `null` if the file doesn't exist.

	$file = $files->get('myfile');

	// If you're fetching a file from a multi upload then you'll have to include the key as well

	$file = $files->get('myfile.0');

The```UploadedFile``` class extends the ```SplFileInfo``` class with the following methods.

| Method name     | Description                                                               |
|-----------------|---------------------------------------------------------------------------|
| getName         | Returns the name of the file that was uploaded                            |
| getReportedSize | Returns the size reported by the client                                   |
| getReportedType | Returns the mime type reported by the client                              |
| hasError        | Returns TRUE if there is an error with the uploaded file and FALSE if not |
| getErrorCode    | Returns the error code associated with the error                          |
| getErrorMessage | Returns a human friendly error message                                    |
| isUploaded      | Returns TRUE if the file is an actual upload and FALSE if not             |
| moveTo          | Moves the file to the specified storage location                          |

> The values reported by the client should not be trusted (e.g. use the ```getSize``` method to retrieve the actual filesize).

--------------------------------------------------------

<a id="server_information"></a>

### Server information

The `getServer` method returns a collection that extends the parameter collection.

	$server = $this->request->getServer();

--------------------------------------------------------

<a id="request_information"></a>

### Request information

The ```referer``` method returns the address that referred the client to the current resource or `null` if there is no referrer.

	// Returns the referrer if there is one and null if not

	$referer = $this->request->referer();

	// Returns the referrer if there is one and the URL to the 'home' route if there isn't one

	$referer = $this->request->referer($this->urlBuilder->toRoute('home'));

The ```ip``` method returns IP of the client that made the request.

	$ip = $this->request->ip();

The ```isAjax``` method returns `true` if the request was made using AJAX and `false` if not.

	$isAjax = $this->request->isAjax();

The ```isSecure``` method returns `true` if the request was made using HTTPS and `false` if not.

	$isSecure = $this->request->isSecure();

The ```isSafe``` method returns `true` if the request method used is considered safe and `false` if not. The methods that are considered safe are ```GET``` and ```HEAD```.

	$isSafe = $this->request->isSafe();

The ```baseURL``` method returns the base URL of the request.

	// A request to 'http://example.org/foo/bar' will return 'http://example.org'

	$baseURL = $this->request->baseURL();

The ```path``` method will return the request path.

	// A request to 'http://example.org/foo/bar' will return '/foo/bar'

	$path = $this->request->path();

The ```language``` method returns the request language (see the ```app/config/application.php``` config file for more information).

	$language = $this->request->language();

The ```languagePrefix``` method returns the language prefix (see the ```app/config/application.php``` config file for more information).

	$languagePrefix = $this->request->languagePrefix();

The ```method``` method returns the request method used to access the resource.

	$method = $this->request->method();

The ```realMethod``` method returns the real request method used to access the resource.

	$method = $this->request->realMethod();

The ```isFaked``` method returns `true` if the request method method has been faked and `false` if not.

	$isFaked = $this->request->isFaked();

The ```username``` method returns a basic HTTP authentication username of `null` if it isn't set.

	$username = $this->request->username();

The ```password``` method returns a basic HTTP authentication password of `null` if it isn't set.

	$password = $this->request->password();

The ```getRoute``` method returns the route that matched the request.

	$route = $this->request->getRoute();
