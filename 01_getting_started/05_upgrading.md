# Upgrading

--------------------------------------------------------

* [Views](#views)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.0.x` to `6.1.x`.

--------------------------------------------------------

<a id="views"></a>

### Views

The `{{$foo || 'Default'}}` and `{{$foo or 'Default'}}` syntax has been deprecated since it will cause unexpected behaviour when printing the output of a ternary expression using the `||` and `or` operators. Use the new `{{$foo, default: 'Default'}}` syntax instead.

The new `default` syntax also has a small change in behaviour. It will print out the value for variables containing `0`, `0.0` and `"0"`.

> Note that the old syntax will not be removed until versio `7.0` but it's still a good idea to update your templates now.