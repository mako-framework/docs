# Error handling

--------------------------------------------------------

* [Custom error handling](#custom_error_handling)
* [Disabling logging of specific exception types](#disabling_logging_of_specific_exception_types)

--------------------------------------------------------

Mako converts all errors to ```ErrorExceptions```. This allows the error handler to display a error page that contains a lot more information than the standard PHP error pages when something bad happens. The default error handler will also log the error if error logging is enabled.

--------------------------------------------------------

<a id="custom_error_handling"></a>

### Custom error handling

You can register custom error handlers for different kinds of exception types using the ```ErrorHandler::handle()``` method. 

A great place to do so is in the ```app/bootstrap.php``` file.

	$errorHandler = $container->get('errorHandler');

	$errorHandler->handle('\PDOException', function($exception)
	{
		// Do some custom logging here
	});

> Your error handler should **not** have a return value unless you want it to stop further error handling.

	$errorHandler->handle('\Exception', function($exception)
	{
		// Do something else here
	});

	$errorHandler->handle('\PDOException', function($exception)
	{
		// Do some custom logging here
	});

> You can register multiple error handlers for any type of error. Note that the last one registered is the first one to get executed.

--------------------------------------------------------

<a id="disabling_logging_of_specific_exception_types"></a>

### Disabling logging of specific exception types

Having error logging enabled can be usefull even when in production, but not all error types are worth logging. You can disable error logging for specific exception types by using the ```disableLoggingFor``` method.

	$errorHandler->disableLoggingFor('\mako\http\routing\PageNotFoundException');

You can also pass an array of exception types.

	$errorHandler->disableLoggingFor
	([
		'\mako\http\routing\PageNotFoundException',
		'\mako\http\routing\MethodNotAllowedException',
	]);

> Remember to use the fully qualified name of the exception types that you want to ignore.