# Sessions

--------------------------------------------------------

* [Usage](#usage)
	- [Data](#usage:data)
	- [Flash data](#usage:flash_data)
	- [Security](#usage:security)
		- [Token](#usage:security:token)
		- [One time tokens](#usage:security:one_time_tokens)
		- [Session id](#usage:security:session_id)
		- [Session destruction](#usage:security:session_destruction)

--------------------------------------------------------

The Mako session library comes with five different session stores by default.

* APCu
* Database
* File
* Null
* Redis

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

#### <a id="usage:data" href="#usage:data">Data</a>

Adding an item to the session is done using the `put` method.

```
$this->session->put('name', $name);
```

> The `mako.flashdata` and `mako.tokens` keys are used by the framework and should not be used to store data.
{.warning}

You can check if an item exists in the session using the `has` method.

```
$this->session->has('name');
```

Getting an item from the session is done using the `get` method.

```
$this->session->get('name');
```

You can also tell the method to return a custom value if the key you're looking for doesn't exist. The default return value for non-existing items is `null`.

```
$this->session->get('name', 'John Doe');
```

You can also get data from the session and replace it using the `getAndPut` method.

```
$this->session->getAndPut('name', $name);
```

It is also possible to retrieve and remove data with a single method call using the `getAndRemove` method.

```
$this->session->getAndRemove('name');
```

If you want to return custom value if the key you're looking for doesn't exist then you can set it using the optional second parameter. The default return value for non-existing items is `null`.

```
$this->session->getAndRemove('name', 'John Doe');
```

Removing data from the session is done using the `remove` method.

```
$this->session->remove('name');
```

If you want to clear all session data then you can use the `clear` method.

```
$this->session->clear();
```

#### <a id="usage:flash_data" href="#usage:flash_data">Flash data</a>

Sometimes you'll want to store temporary data that should expire after the next request (e.g., error and status messages). For this you can use the `putFlash` method.

```
$this->session->putFlash('success', 'The article has successfully been deleted!');
```

Retrieving flash data is done using the `getFlash` method.

```
$data = $this->session->getFlash('success');
```

You can also tell the method to return a custom value if the key you're looking for doesn't exist. The default return value for non-existing items is `null`.

```
$data = $this->session->getFlash('success', 'Some other message.');
```

You can check if a flash item exists in the session using the `hasFlash` method.

```
$this->session->hasFlash('success');
```

You can remove flash data using the `removeFlash` method. This is usually not needed as flash data expires after one request.

```
$this->session->removeFlash('success');
```

Sometimes you'll want to extend the lifetime of the flash data by one request. This can be done using the `reflash` method.

```
$this->session->reflash();
```

You can also choose to reflash only a set of keys if you don't want to reflash all the flash data.

```
$this->session->reflash(['success', 'error']);
```

#### <a id="usage:security" href="#usage:security">Security</a>

##### <a id="usage:security:token" href="#usage:security:token">Token</a>

The `getToken` returns a token that can be used in forms and AJAX requests to prevent [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

```
$token = $this->session->getToken();
```

The `validateToken` method allows you to validate a token. It will return `true` if the token is valid and `false` if not. You can also validate tokens using the `token` rule of the [validator library](:base_url:/docs/:version:/learn-more:validation).

```
$valid = $this->session->validateToken($token);
```

The `regenerateToken` method lets you generates a new token.

```
$token = $this->session->regenerateToken();
```

> Mako will automatically generate a new token upon a successful login and logout when using the [Gatekeeper](:base_url:/docs/:version:/security:gatekeeper) authentication library.

##### <a id="usage:security:one_time_tokens" href="#usage:security:one_time_tokens">One time tokens</a>

```
$token = $this->session->regenerateToken();
```

The `generateOneTimeToken` method allows you to generate a one time token that can be used in forms to prevent [CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery).

```
$token = $this->session->generateOneTimeToken();
```

The `validateOneTimeToken` method allows you to validate a one time token. It will return `true` if the token is valid and `false` if not. You can also validate tokens using the `one_time_token` rule of the [validator library](:base_url:/docs/:version:/learn-more:validation).

One time tokens that have been validated once are no longer considered valid so they are not well suited for use with AJAX requests.

```
$valid = $this->session->validateOneTimeToken($token);
```

> Note that only the last 20 one time tokens that have been generated during a session are valid.

##### <a id="usage:security:session_id" href="#usage:security:session_id">Session id</a>

Retrieving the session id is done by using the `getId` method.

```
$id = $this->session->getId();
```

Regenerating the session id can be done by using the `regenerateId` method. A general rule of thumb is to regenerate the session id each time the access level of the user changes.

```
$this->session->regenerateId();
```

You can tell it to keep the data associated with the old session id

```
$this->session->regenerateId(true);
```

> Mako will automatically regenerate the session id upon a successful login and logout when using the [Gatekeeper](:base_url:/docs/:version:/security:gatekeeper) authentication library.

##### <a id="usage:security:session_destruction" href="#usage:security:session_destruction">Session destruction</a>

To destroy a session use the `destroy` method.

```
$this->session->destroy();
```
