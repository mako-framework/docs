# Upgrading

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `8.0.x` to `8.1.x`. 

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

#### String helpers

The `Str::camel2underscored()` and `Str::underscored2camel()` methods have been deprecated and replaced by the `Str::camelToSnake()` and `Str::snakeToCamel()` methods.
