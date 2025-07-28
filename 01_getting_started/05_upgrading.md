# Upgrading

* [CLI](#cli)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `11.1.x` to `11.2.x`.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### <a id="cli" href="#cli">CLI</a>

The `Command::question()` method has been deprecated and will be removed in `Mako 12`. You can replace it with the new `Command::input()` method.

The `Cursor::beginningOfLine()` method has been deprecated and will be removed in `Mako 12`. You can use the `Cursor::moveToBeginningOfLine()` method instead.