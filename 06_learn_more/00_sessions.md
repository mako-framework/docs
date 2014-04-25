# Sessions

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The Mako session library comes with three different session stores by default.

* Database
* File
* Redis

--------------------------------------------------------

<a id="usage"></a>

### Usage

Adding an item to the session is done using the ```put``` method.

	$this->session->put('name', $name);

Getting an item from the session is done using the ```get``` method.

	$this->session->get('name');

	// The get method allows you to return a default value if the key you're looking for doesn't exist
	// The default return value for non-existing items is NULL

	$this->session->get('name', 'John Doe');

You can check if an item exists in the session using the ```has``` method.

	$this->session->has('name');

Removing data from the session is done using the ```remove``` method.

	$this->session->remove('name');

Sometimes you'll want to store temporary data that should expire after the next request (e.g., error and status messages). For this you can use the ```flash``` method.

	$this->session->flash('success', 'The article has successfully been deleted!');

Sometimes you'll want to extend the lifetime of the flashdata. This can be done using the ```reflash``` method.

	$this->session->reflash();

	// You can also choose to reflash only a set of keys

	$this->session->reflash(['success', 'error']);

Retrieving flash data is done using the ```flash``` method as well. The method will return FALSE if the requested flash data doesn't exist.

	$data = $this->session->flash('success');

The ```generateToken``` allows you to generate a token that can be used in forms to prevent [CSRF](http://en.wikipedia.org/wiki/Cross-site_request_forgery).

	$token = $this->session->generateToken();

The ```validateToken``` method allows you to validate a token. It will return TRUE if the token is valid and FALSE if not. You can also validate tokens using the ```token``` rule of the [validator library](:base_url:/docs/:version:/learn-more:validation).

	$valid = $this->session->validateToken($token);

> Only the last 20 tokens that have been generated during a session are valid. Tokens that have been validated once are no longer considered valid.

Retrieving the session id is done by using the ```getId``` method.

	$id = $this->session->getId();

Regenerating the session id can be done by using the ```regenerateId``` method. A general rule of thumb is to regenerate the session id each time the access level of the user changes.

	$this->session->regenerateId();

	// You can tell it to keep the data associated with the old session id

	$this->session->regenerateId(true);

To destroy a session use the ```destroy``` method.

	$this->session->destroy();

If you want to clear all session data without destroying the session then you can use the ```clear``` method.

	$this->session->clear();