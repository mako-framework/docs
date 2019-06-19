# Upgrading

--------------------------------------------------------

* [Views](#views)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.1.x` to `6.2.x`. You should also take a look at the changelog for a full list of changes to internal classes and methods.

--------------------------------------------------------

<a id="views"></a>

### Views

The `{{$foo || 'Default'}}` syntax has been deprecated since it will cause unexpected behaviour when printing the output of a ternary expression using the `||` operator. Use the `{{$foo or 'Default'}}` syntax instead.