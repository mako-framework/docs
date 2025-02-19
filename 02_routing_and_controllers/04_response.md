# Response

--------------------------------------------------------

* [Basics](#basics)
* [Cookies](#cookies)
* [Headers](#headers)
* [Caching and compression](#caching_and_compression)

--------------------------------------------------------

The response class helps you build a HTTP response.

An instance of the response class is always available in all controller classes. It is also easily made available in [route closures](:base_url:/docs/:version:/routing-and-controllers:routing#basics).

--------------------------------------------------------

### <a id="basics" href="#basics">Basics</a>

The `setBody` method allows you to set the response body. This is not normally needed as the framework automatically sets the response body to the return value of your controller/route action. It can; however, be useful if you want to alter the response body in [middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware).

```
$this->response->setBody('Hello, world!');
```

The `getBody` method returns the response body.

```
$responseBody = $this->response->getBody();
```

The `setType` method lets you define the response type as well as the response charset. The default response type is `text/html` and the default charset is `UTF-8`. The default charset is defined in your `app\config\application.php` config file.

```
$this->response->setType('application/json');

// You can also override the default charset

$this->response->setType('application/json', 'iso-8859-1');
```

The `getType` method returns the current response type.

```
$responseType = $this->response->getType();
```

The `setCharset` method lets you set the response charset. The default charset is `UTF-8`and it is defined in your `app\config\application.php` config file.

```
$this->response->setCharset('iso-8859-1');
```

The `getCharset` method returns the current response charset.

```
$responseCharset = $this->response->getCharset();
```

The `setStatus` method lets you set the response status.

```
// You can set the status using the Status enum

$this->response->setStatus(Status::NOT_FOUND);

// Or by passing a valid status code integer

$this->response->setStatus(404);
```

The `getStatus` method returns the current status code.

```
$responseStatus = $this->response->getStatus();
```

--------------------------------------------------------

### <a id="cookies" href="#cookies">Cookies</a>

The `cookies` property is a cookie collection.

```
$cookies = $this->response->cookies;

// You can also use the "getCookies" method

$cookies = $this->response->getCookies();
```

The `add` method adds a cookie to the response.

```
// Sets a cookie that expires when the browser closes

$cookies->add('name', 'value');

// Sets a cookie that expires after 1 hour

$cookies->add('name', 'value', 3600);

// You can also set the `path`, `domain`, `secure`, `httponly` and `samesite` options using the fourth parameter

$cookies->add('name', 'value', 3600, ['path' => '/mydir', 'domain' => '.example.org']);
```

> If you want to set signed cookies then you'll have to use the `addSigned` method. The benefit of using signed cookies is that they can't be tampered with on the client side.

The `delete` method allows you to delete both normal and signed cookies from the client.

```
$cookies->delete('name');

// You can also set the `path`, `domain`, `secure`, `httponly` and `samesite` options using the fourth parameter

$cookies->delete('name', ['path' => '/mydir', 'domain' => '.example.org']);
```

The cookie collection also includes the following methods in addition to the ones shown in the examples above.

| Method                                      | Description                                                                    |
|---------------------------------------------|--------------------------------------------------------------------------------|
| setOptions($options)                        | Allows you to override the default cookie options                              |
| addRaw($name, $value, $ttl, $options)       | Adds a raw unsigned cookie                                                     |
| addRawSigned($name, $value, $ttl, $options) | Adds a raw signed cookie                                                       |
| has($name)                                  | Returns `true` if the response includes the cookie and `false` if not          |
| remove($name)                               | Removes the cookie from the response                                           |
| clear()                                     | Removes all cookies from the response                                          |
| clearExcept($cookies)                       | Removes all cookies except for the ones in the provided list from the response |
| all()                                       | Returns an array containing all the cookies                                    |

--------------------------------------------------------

### <a id="headers" href="#headers">Headers</a>

The `headers` property is a header collection.

```
$headers = $this->response->headers;

// You can also use the "getHeaders" method

$headers = $this->response->getHeaders();
```

The `add` method adds a header to your response.

```
$headers->add('X-My-Header', 'value');
```

The header collection also includes the following methods in addition to the ones shown in the examples above.

| Method                  | Description                                                                    |
|-------------------------|--------------------------------------------------------------------------------|
| has($name)              | Returns `true` if the response includes the header and `false` if not          |
| hasValue($name, $value) | Returns `true` if the header has the specified value and `false` if not        |
| remove($name)           | Removes the header from the response                                           |
| clear()                 | Removes all headers from the response                                          |
| clearExcept($headers)   | Removes all headers except for the ones in the provided list from the response |
| all()                   | Returns an array containing all the headers                                    |

--------------------------------------------------------

### <a id="caching_and_compression" href="#caching_and_compression">Caching and compression</a>

You can enable `ETag` caching using the `enableCaching` method. Doing so can save bandwidth as the response body will only be sent if it has been modified since the last request.

```
$this->response->enableCaching();
```

The `disableCaching` method disables `ETag` caching if it has been enabled.

```
$this->response->disableCaching();
```

The `isCacheable` method returns `true` if the response in its current state is considered cacheable and `false` if not.

```
$isCacheable = $this->response->isCacheable();
```

> Note that `ETag` caching will only be used if the response is considered cacheable.

The `enableCompression` method enables output compression. This will save you bandwidth in exchange for a slight bump in CPU usage.

```
$this->response->enableCompression();
```

The `disableCompression` method disables output compression if it has been enabled.

```
$this->response->disableCompression();
```

You can specify a custom compression handler instead of using the default `ob_gzhandler` by using the `setCompressionHandler` method. This allows for greater flexibility in choosing different compression algorithms, such as gzip, deflate, or Brotli, based on client support and your application’s requirements.

When implementing a custom [compression handler](https://www.php.net/manual/en/function.ob-start.php), you must:

1) Check that the client supports the compression method by checking the `Accept-Encoding` header.
2) Compress the output using your compression handler.
3) Set the correct `Content-Encoding` header so the client knows how to decode the response.
4) Include a `Vary: Accept-Encoding` header to ensure proper caching behavior by proxies and CDNs.

The best place to implement this functionality is within a custom [middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware). Middleware allows you to intercept and modify HTTP requests and responses before they reach your application logic or before being sent to the client.

```
$this->response->setCompressionHandler(static function (string $buffer, int $phase): string {
    // Your custom compression code goes here
});
```

The `getCompressionHandler` method returns the currently active compression handler being used for output buffering. By default, this is `ob_gzhandler`, which automatically compresses output using gzip or deflate, depending on what the client supports.

```
$compressionHandler = $this->response->getCompressionHandler();
```