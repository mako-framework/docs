# Upgrading

--------------------------------------------------------

* [Error handling](#error_handling)
* [Views](#views)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.0.x` to `6.1.x`.

--------------------------------------------------------

<a id="error_handling"></a>

### Error handling

The `ErrorHandler::disableLoggingFor()` method has been deprecated. Use the new `ErrorHandler::dontLog()` method or the new `application.error_handler.dont_log` config key instead.

> Note that the method will not be removed until version `7.0` but it's still a good idea to update your code now.

--------------------------------------------------------

<a id="views"></a>

### Views

The `{{$foo || 'Default'}}` and `{{$foo or 'Default'}}` syntax has been deprecated since it will cause unexpected behaviour when printing the output of a ternary expression using the `||` and `or` operators. Use the new `{{$foo, default: 'Default'}}` syntax instead.

The new `default` syntax also has a small change in behaviour. It will print out the value for variables containing `0`, `0.0` and `"0"`.

You can use the following regex to find your echo tags that still use the deprecated syntax:

```
\{\{([^view:][^}]+)((\|\|){1}|\s+(or)\s+)([^}]+)\}\}
```
{.language-regex}

> Note that the old syntax will not be removed until version `7.0` but it's still a good idea to update your templates now.