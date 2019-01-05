# Configuration

--------------------------------------------------------

* [Config files](#config_files)
* [Environment aware configuration](#environment_aware_configuration)
	- [Setting the environment](#environment_aware_configuration:setting_the_environment)
* [Package configuration](#package_configuration)

--------------------------------------------------------

The configuration of the Mako core is done in the `app/init.php` file. This is where you set the error reporting level and define the paths to the application and vendor directories.

All of the remaining framework configuration is done by editing the files that are located in the `app/config` directory.

--------------------------------------------------------

<a id="config_files"></a>

### Config files

Mako config files are just simple arrays:

```
<?php

return
[
	'key_1' => 'value',
	'key_2' => 'value',
];
```

And loading a config file is done by using the `get` method.

```
$config = $this->config->get('redis'); // Loads the redis.php file
```

You can also fetch config items using `dot notation`.

```
$default = $this->config->get('redis.default');
```

It is also possible to override settings or add new configurations at runtime:

```
// Adds a new Crypto configuration named "user" that you can
// use when creating a Crypto instance "Crypto::instance('user');"

$this->config->set('crypto.configurations.user',
[
	'library' => 'openssl',
	'cipher'  => 'AES-256-OFB',
	'key'     => 'ksMGBr_yR>=IiRicJFUhD4XlRnE%|11mvRGNJsD',
]);
```

Removing the custom configuration is done using the `remove` method:

```
$this->config->remove('crypto.configurations.user');
```

> Setting configuration at runtime is not always possible. Some components such as the connections managers (database, redis, etc...) will cache the settings once they get loaded. You can override them using their `addConfiguration` and `removeConfiguration` methods instead.
{.warning}

--------------------------------------------------------

<a id="environment_aware_configuration"></a>

### Environment aware configuration

Mako supports environment aware configuration. This means that you can have separate configuration files for your different environments. All you have to do is create a subdirectory with the name of your environment in the `app/config` directory and copy the environment specific files into it.

<a id="environment_aware_configuration:setting_the_environment"></a>

#### Setting the environment

Setting the environment in Apache:

```
SetEnv MAKO_ENV dev
```
{.language-apacheconf}

Setting the environment in Nginx:

```
fastcgi_param MAKO_ENV dev;
```
{.language-nginx}

Setting the environment in a linux/unix shell:

```
export MAKO_ENV=dev # for Bourne, bash, and related shells
setenv MAKO_ENV=dev # for csh and related shells
```
{.language-none}

You can also manually set the environment in the [CLI](:base_url:/docs/:version:/command-line:basics) using the env option.

```
php reactor <command> --env=dev
```
{.language-none}

--------------------------------------------------------

<a id="package_configuration"></a>

### Package configuration

Check out the [package documentation](:base_url:/docs/:version:/packages:packages#configuration_i18n_and_views) for more information regarding package configuration.
