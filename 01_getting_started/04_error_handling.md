# Error handling

--------------------------------------------------------

* [Custom error handling](#custom_error_handling)

--------------------------------------------------------

Mako converts all errors to ```ErrorExceptions```. This allows the error handler to display a error page that contains a lot more information than the standard PHP error pages when something bad happens. The default error handler will also log the error if error logging is enabled.

--------------------------------------------------------

<a id="custom_error_handling"></a>

### Custom error handling

You can register custom error handlers for different kinds of exception types using the ```Application::handle()``` method. 

A great place to do so is in the ```app/bootstrap.php``` file.

	$app->handle('\PDOException', function($exception)
	{
		// Do some custom logging here
	});

> Your error handler should **not** have a return value unless you want it to stop further error handling.

	$app->handle('\Exception', function($exception)
	{
		// Do something else here
	});

	$app->handle('\PDOException', function($exception)
	{
		// Do some custom logging here
	});

> You can register multiple error handlers for any type of error. Note that the last one registered is the first one to get executed.