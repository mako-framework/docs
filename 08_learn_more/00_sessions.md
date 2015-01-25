# Sessions

--------------------------------------------------------

* [Usage](#usage)
	- [Data](#usage:data)
	- [Flash data](#usage:flash_data)
	- [Security](#usage:security)
		- [Token](#usage:security:token)
		- [One time tokens](#usage:security:one_time_tokens)
		- [Session id](#usage:security:session_id)
		- [Session data](#usage:security:session_data)

--------------------------------------------------------

The Mako session library comes with four different session stores by default.

* Database
* File
* Null
* Redis

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:data"></a>

#### Data

Adding an item to the session is done using the ```put``` method.

	$this->session->put('name', $name);

> The ```mako.flashdata``` and ```mako.tokens``` keys are used by the framework and should not be used to store data. 

Getting an item from the session is done using the ```get``` method.

	$this->session->get('name');

You can also tell the method to return a custom value if the key you're looking for doesn't exist. The default return value for non-existing items is NULL.

	$this->session->get('name', 'John Doe');

You can check if an item exists in the session using the ```has``` method.

	$this->session->has('name');

Removing data from the session is done using the ```remove``` method.

	$this->session->remove('name');

<a id="usage:flash_data"></a>

#### Flash data

Sometimes you'll want to store temporary data that should expire after the next request (e.g., error and status messages). For this you can use the ```putFlash``` method.

	$this->session->putFlash('success', 'The article has successfully been deleted!');

Retrieving flash data is done using the ```getFlash``` method.

	$data = $this->session->getFlash('success');

You can also tell the method to return a custom value if the key you're looking for doesn't exist. The default return value for non-existing items is NULL.

	$data = $this->session->getFlash('success', 'Some other message.');

You can check if a flash item exists in the session using the ```hasFlash``` method.

	$this->session->hasFlash('success');

You can remove flash data using the ```removeFlash``` method. This is usually not needed as flash data expirtes after one request.

	$this->session->removeFlash('success');

Sometimes you'll want to extend the lifetime of the flash data by one request. This can be done using the ```reflash``` method.

	$this->session->reflash();

You can also choose to reflash only a set of keys if you don't want to reflash all the flash data.

	$this->session->reflash(['success', 'error']);

<a id="usage:security"></a>

#### Security

<a id="usage:security:token"></a>

##### Token

The ```getToken``` returns a token that can be used in forms and AJAX requests to prevent [CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery).

	$token = $this->session->getToken();

The ```validateToken``` method allows you to validate a token. It will return TRUE if the token is valid and FALSE if not. You can also validate tokens using the ```token``` rule of the [validator library](:base_url:/docs/:version:/learn-more:validation).

	$valid = $this->session->validateToken($token);

The ```regenerateToken``` method lets you generates a new token. 

	$token = $this->session->regenerateToken();

> Mako will automatically generate a new token upon a succesful login and logout when using the [Gatekeeper](:base_url:/docs/:version:/security:authentication) authentication library.

<a id="usage:security:one_time_tokens"></a>

##### One time tokens

	$token = $this->session->regenerateToken();

The ```generateOneTimeToken``` method allows you to generate a one time token that can be used in forms to prevent [CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery).

	$token = $this->session->generateOneTimeToken();

The ```validateOneTimeToken``` method allows you to validate a one time token. It will return TRUE if the token is valid and FALSE if not. You can also validate tokens using the ```one_time_token``` rule of the [validator library](:base_url:/docs/:version:/learn-more:validation). 

One time tokens that have been validated once are no longer considered valid so they are not well suited for use with AJAX requests.

	$valid = $this->session->validateOneTimeToken($token);

> Note that only the last 20 one time tokens that have been generated during a session are valid.

<a id="usage:security:session_id"></a>

##### Session id

Retrieving the session id is done by using the ```getId``` method.

	$id = $this->session->getId();

Regenerating the session id can be done by using the ```regenerateId``` method. A general rule of thumb is to regenerate the session id each time the access level of the user changes.

	$this->session->regenerateId();

You can tell it to keep the data associated with the old session id

	$this->session->regenerateId(true);

> Mako will automatically regnerate the session id upon a succesful login and logout when using the [Gatekeeper](:base_url:/docs/:version:/security:authentication) authentication library.

<a id="usage:security:session_data"></a>

##### Session data

To destroy a session use the ```destroy``` method.

	$this->session->destroy();

If you want to clear all session data without destroying the session then you can use the ```clear``` method.

	$this->session->clear();