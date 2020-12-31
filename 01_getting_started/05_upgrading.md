# Upgrading

--------------------------------------------------------

* [Commands](#commands)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `7.0.x` to `7.1.x`. The release does not contain any breaking changes but there are a couple of optioanl new features that you can start using.

--------------------------------------------------------

<a id="commands"></a>

### Commands

Mako `7.1` has added support for automatic registration of [reactor commands](:base_url:/docs/:version:/command-line:commands). All you have to do is to replace your `commands` config key with the [new](https://github.com/mako-framework/app/blob/e7a7c24456e16f187fce5fff07c402898f8ec97f/app/config/application.php#L94) `commands_directory` key and to add the new `$command` property to your commands as documented in the [commands documentation](:base_url:/docs/:version:/command-line:commands).