# Sessions

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The session class overrides PHP's standard session handler. This allows you to use the ```$_SESSION``` array as well as the methods offered by the session class. The supported handlers are:

* Database
* File
* Native
* Redis

> The native handler will use the default PHP session handler that has been configured in php.ini.

--------------------------------------------------------

<a id="usage"></a>

### Usage

If you want to use the ```$_SESSION``` array directly or the non-static interface of the session class then you need to call the instance method.

	$session = Session::instance();

Adding an item to the session is done using the ```remember``` method.

	$session->remember('name', $name);

	// Or if you're using the static interface

	Session::remember('name', $name);

Getting an item from the session is done using the ```get``` method.

	$session->get('name');

	// Or if you're using the static interface

	Session::get('name');

	// The get method allows you to return a default value if the key you're looking for doesn't exist
	// The default return value for non-existing items is NULL

	$session->get('name', 'John Doe');

You can check if an item exists in the session using the ```has``` method.

	$session->has('name');

	// Or if you're using the static interface

	Session::has('name');

Removing data from the session is done using the ```forget``` method.

	$session->forget('name');

	// Or if you're using the static interface

	Session::forget('name');

Sometimes you'll want to store temporary data that should expire after the next request (e.g., error and status messages). For this you can use the ```flash``` method.

	$session->flash('success', 'The article has successfully been deleted!');

	// Or if you're using the static interface

	Session::flash('success', 'The article has successfully been deleted!');

Sometimes you'll want to extend the lifetime of the flashdata. This can be done using the ```reflash``` method.

	$session->reflash();

	// Or if you're using the static interface

	Session::reflash();

Retrieving flash data is done using the ```flash``` method as well. The method will return FALSE if the requested flash data doesn't exist.

	$data = $session->flash('success');

	// Or if you're using the static interface

	$data = Session::flash('success');

Retrieving the session id is done by using the ```id``` method.

	$id = $session->id();

	// Or if you're using the static interface

	$id = Session::id();

Regenerating the session id can be done by using the ```regenerate``` method. A general rule of thumb is to regenerate the session id each time the access level of the user changes.

	$session->regenerate();

	// Or if you're using the static interface

	Session::regenerate();

To destroy a session use the ```destroy``` method.

	$session->destroy();

	// Or if you're using the static interface

	Session::destroy();

If you want to clear all session data without destroying the session then you can use the ```clear``` method.

	$session->clear();

	// Or if you're using the static interface

	Session::clear();