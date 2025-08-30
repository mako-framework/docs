# Upgrading

* [Crypto](#crypto)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `11.3.x` to `11.4.x`.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### <a id="crypto" href="#crypto">Crypto</a>

The `CryptoManager::getEncrypter()` method has been deprecated and will be removed in `Mako 12`. You can replace it with the `CryptoManager::getInstance()` method.