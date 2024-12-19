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

### <a id="request_data" href="#request_data">Accessing data</a>

#### <a id="request_data:query_string_post_data_and_parsed_body" href="#request_data:query_string_post_data_and_parsed_body">Query string, post data and parsed body</a>

The `query` property is a parameter collection containing query string data.

```
$queryString = $this->request->query;

// You can also use the "getQuery" method

$queryString = $this->request->getQuery();
```

The `get` method of the parameter collection returns the value of a parameter and `null` if it doesn't exist.

```
$value = $queryString->get('foo');

// You can also specify a custom default return value

$value = $queryString->get('foo', false);
```

The `post` property is a parameter collection containing post data.

```
$post = $this->request->post;

// You can also use the "getPost" method

$post = $this->request->getPost();
```

The `body` property is a parameter collection containing data from a parsed request body. The property currently supports `application/x-www-form-urlencoded` and `application/json` request bodies.

```
$body = $this->request->body;

// You can also use the "getBody" method

$body = $this->request->getBody();
```

The `data` property is a parameter collection containing the data corresponding to the request method (query string for `GET` requests, post data or a parsed body for `POST` requests and the parsed body for any other type of request).

```
$data = $this->request->data;

// You can also use the "getData" method

$data = $this->request->getData();
```

The parameter collections also includes the following methods in addition to the ones shown in the examples above.

| Method                             | Description                                                                 |
|------------------------------------|-----------------------------------------------------------------------------|
| all()                              | Returns an array containing all the parameters                              |
| add($name, $value)                 | Adds a parameter                                                            |
| has($name)                         | Returns `true` if the collection has the parameter and `false` if not       |
| remove($name)                      | Removes a parameter                                                         |
| whitelisted($keys, $defaults = []) | Returns a whitelisted array of parameters                                   |
| blacklisted($keys, $defaults = []) | Returns an array of parameters where the blacklisted keys have been removed |

#### <a id="request_data:raw_request_body" href="#request_data:raw_request_body">Raw request body</a>

The `rawBody` property contains the raw request body as a string.

```
$body = $this->request->rawBody;

// You can also use the "getRawBody" method.

$body = $this->request->getRawBody();
```

The `getRawBodyAsStream` method returns the raw request body as a stream. This can be useful if you want to minimize the memory footprint of your application while handling large request bodies.

```
$body = $this->request->getRawBodyAsStream();

$storageStream = fopen($this->app->getPath() . '/storage/uploads/' . $filename, 'w');

$fileSize = stream_copy_to_stream($body, $storageStream);
```

--------------------------------------------------------

### <a id="cookies" href="#cookies">Cookies</a>

The `cookies` property is a cookie collection that allows to you access both signed and unsigned cookies.

```
$cookies = $this->request->cookies;

// You can also use the "getCookies" method

$cookies = $this->request->getCookies();
```

The `get` method of the cookie collection returns the value of a cookie and `null` if it doesn't exist.

```
$value = $cookies->get('foobar');

// You can also define a custom default return value

$value = $cookies->get('foobar', false);
```

> If you want to read a signed cookie then you'll have to use the `getSigned` method. The benefit of using signed cookies is that they can't be tampered with on the client side.

The cookie collection also includes the following methods in addition to the ones shown in the examples above.

| Method                   | Description                                            |
|--------------------------|--------------------------------------------------------|
| all()                    | Returns an array containing all the cookies            |
| add($name, $value)       | Adds a cookie                                          |
| addSigned($name, $value) | Adds a signed cookie                                   |
| has($name)               | Returns `true` if the cookie exists and `false` if not |
| remove($name)            | Removes a cookie                                       |

--------------------------------------------------------

### <a id="headers" href="#headers">Headers</a>

The `headers` property is a header collection.

```
$headers = $this->request->headers;

// You can also use the "getHeaders" method

$headers = $this->request->getHeaders();
```

The `get` method of the header collection returns the value of the header an `null` it it isn't set.

```
$value = $headers->get('content-type');

// You can also define a custom default return value

$value = $headers->get('content-type', 'text/plain');
```

The `getAcceptableContentTypes` method of the header collection returns an array of acceptable content types in descending order of preference.

```
$contentTypes = $headers->getAcceptableContentTypes();
```

The `getAcceptableLanguages` method of the header collection returns an array of acceptable languages in descending order of preference.

```
$languages = $headers->getAcceptableLanguages();
```

The `getAcceptableCharsets` method of the header collection returns an array of acceptable character sets in descending order of preference.

```
$charsets = $headers->getAcceptableCharsets();
```

The `getAcceptableEncodings` method of the header collection returns an array of acceptable encodings in descending order of preference.

```
$charsets = $headers->getAcceptableEncodings();
```

The header collection also includes the following methods in addition to the ones shown in the examples above.

| Method             | Description                                            |
|--------------------|--------------------------------------------------------|
| all()              | Returns an array containing all the headers            |
| add($name, $value) | Adds a header                                          |
| has($name)         | Returns `true` if the header exists and `false` if not |
| remove($name)      | Removes a header                                       |
| getBearerToken()   | Returns the bearer token or `null` if there isn't one  |

--------------------------------------------------------

### <a id="files" href="#files">Files</a>

The `files` property is a file collection that extends the parameter collection.

```
$files = $this->request->files;

// You can also use the "getFiles" method

$files = $this->request->getFiles();
```

The `get` method of the file collection returns a `UploadedFile` instance or `null` if the file doesn't exist.

```
$file = $files->get('myfile');

// If you're fetching a file from a multi upload then you'll have to include the key as well

$file = $files->get('myfile.0');
```

The `UploadedFile` class extends the [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) class with the following methods.

| Method                | Description                                                                   |
|-----------------------|-------------------------------------------------------------------------------|
| getReportedFilename() | Returns the name of the file that was uploaded                                |
| getReportedSize()     | Returns the size reported by the client                                       |
| getReportedMimeType() | Returns the mime type reported by the client                                  |
| hasError()            | Returns `true` if there is an error with the uploaded file and `false` if not |
| getErrorCode()        | Returns the error code associated with the error                              |
| getErrorMessage()     | Returns a human friendly error message                                        |
| isUploaded()          | Returns `true` if the file is an actual upload and `false` if not             |
| moveTo($path)         | Moves the file to the specified storage location                              |

> The mime type and size values reported by the client should not be trusted (e.g. use the `getSize` method to retrieve the actual file size).
{.warning}

--------------------------------------------------------

### <a id="server_information" href="#server_information">Server information</a>

The `server` property is a collection that extends the parameter collection.

```
$server = $this->request->server;

// You can also use the "getServer" method

$server = $this->request->getServer();
```

--------------------------------------------------------

### <a id="request_information" href="#request_information">Request information</a>

The `getContentType` method returns the content type of the request body.

```
$contentType = $this->request->getContentType();
```

The `getReferrer` method returns the address that referred the client to the current resource or `null` if there is no referrer.

```
// Returns the referrer if there is one and "null" if not

$referrer = $this->request->getReferrer();

// Returns the referrer if there is one and the URL to the 'home' route if there isn't one

$referrer = $this->request->getReferrer($this->urlBuilder->toRoute('home'));
```

The `getIp` method returns IP of the client that made the request.

```
$ip = $this->request->getIp();
```

The `isAjax` method returns `true` if the request was made using AJAX and `false` if not.

```
$isAjax = $this->request->isAjax();
```

The `isSecure` method returns `true` if the request was made using HTTPS and `false` if not.

```
$isSecure = $this->request->isSecure();
```

The `isSafe` method returns `true` if the request method used is considered safe and `false` if not. The methods that are considered safe are `GET`, `HEAD`, `OPTIONS` and `TRACE`.

```
$isSafe = $this->request->isSafe();
```

The `isCacheable` method returns `true` if the request method used is considered cacheable and `false` if not. The methods that are considered cacheable are `GET` and `HEAD`.

```
$isCacheable = $this->request->isCacheable();
```

The `isIdempotent` method returns `true` if the request method used is considered idempotent and `false` if not. The methods that are considered idempotent are `DELETE`, `GET`, `HEAD`, `OPTIONS`, `PUT` and `TRACE`.

```
$isIdempotent = $this->request->isIdempotent();
```

The `getBasePath` method returns the base path of the request. A request to `http://example.org/foo/index.php` will return `/foo` while a request to `http://example.org/index.php` will return an empty string.

```
$basePath = $this->request->getBasePath();
```

The `getBaseURL` method returns the base URL of the request. A request to `http://example.org/foo/bar` will return `http://example.org`.

```
$baseURL = $this->request->getBaseURL();
```

The `getPath` method will return the request path. A request to `http://example.org/index.php/foo/bar` or `http://example.org/foo/bar` will return `/foo/bar`.

```
$path = $this->request->getPath();
```

The `getLanguage` method returns the request language (see the `app/config/application.php` config file for more information).

```
$language = $this->request->getLanguage();
```

The `getLanguagePrefix` method returns the language prefix (see the `app/config/application.php` config file for more information).

```
$languagePrefix = $this->request->getLanguagePrefix();
```

The `getMethod` method returns the request method used to access the resource.

```
$method = $this->request->getMethod();
```

The `getRealMethod` method returns the real request method used to access the resource.

```
$method = $this->request->getRealMethod();
```

The `isFaked` method returns `true` if the request method method has been faked and `false` if not.

```
$isFaked = $this->request->isFaked();
```

The `getUsername` method returns a basic HTTP authentication username of `null` if it isn't set.

```
$username = $this->request->getUsername();
```

The `getPassword` method returns a basic HTTP authentication password of `null` if it isn't set.

```
$password = $this->request->getPassword();
```

The `getRoute` method returns the route that matched the request.

```
$route = $this->request->getRoute();
```
