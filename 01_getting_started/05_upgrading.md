# Upgrading

* [Cache](#cache)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 11.0.x to 11.1.x.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### <a id="cache" href="#cache">Cache</a>

The `WinCache` cache store has been deprecated and will be removed in Mako 12. As a result, it's recommended that you transition to one of the other available cache stores to ensure compatibility with future versions of Mako.