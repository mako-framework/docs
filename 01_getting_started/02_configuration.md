# Configuration

--------------------------------------------------------

* [Config files](#config_files)
	- [Security First](#security_first)
* [Environment aware configuration](#environment_aware_configuration)
	- [Setting the environment](#environment_aware_configuration:setting_the_environment)
	- [Environment variables](#environment_aware_configuration:environment_variables)
* [Package configuration](#package_configuration)

--------------------------------------------------------

The configuration of the Mako core is done in the `app/init.php` file. This is where you set the error reporting level and define the paths to the application and vendor directories.

All of the remaining application configuration is done by editing the files that are located in the `app/config` directory.

--------------------------------------------------------

### <a id="config_files" href="#config_files">Config files</a>

Mako config files are just simple arrays. In addition to the core config files that are created, you can create your own custom config files:

```
<?php

// app/config/redis.php
return
[
	'key_1' => 'value',
	'key_2' => 'value',
];
```

And loading a config file is done by using the `get` method.

```
$config = $this->config->get('redis'); // Loads the app/config/redis.php file
```

You can also fetch config items using `dot notation`. If we are using the example above, you can fetch `key_1` in this manner:

```
$default = $this->config->get('redis.key_1');
```

It is also possible to override settings or add new configurations at runtime:

```
// Adds a new Crypto configuration named "user" that you can
// use when creating a Crypto instance "Crypto::getInstance('user');"

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

#### <a id="security_first" href="#security_first">Security First</a>

Many developers have made mistakes with committing config files containing sensitive information. In some cases those repositories are made public and there are credential sniffing bots that find exposed credentials. When you are dealing with Mako application, please take caution into what files you commit to the repository and which files you add to your `.gitignore` or equivalent ignore file for your VCS. 

> Any configuration files that hold an API key, a username/password, an encryption key or any other type of sensitive data should _not_ be committed to your repository!
{.danger}

--------------------------------------------------------

### <a id="environment_aware_configuration" href="#environment_aware_configuration">Environment aware configuration</a>

Mako supports environment aware configuration. This means that you can have separate configuration files for your different environments. All you have to do is create a subdirectory with the name of your environment in the `app/config` directory and copy the environment specific files into it.

#### <a id="environment_aware_configuration:setting_the_environment" href="#environment_aware_configuration:setting_the_environment">Setting the environment</a>

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

Using the above methods to set up the `dev` environment, it will now read your configurations from `app/config/dev/`. If you changed the environment to `prod` it will pull from `app/config/prod/` instead.

#### <a id="environment_aware_configuration:environment_variables" href="#environment_aware_configuration:environment_variables">Environment variables</a>

You can also use the `mako\env` function to fetch environment variables. This is a good idea for senstive configuration values such as credentials, encryption keys, etc...

```
[
	'username'    => mako\env('USERNAME'),
	'password'    => mako\env('PASSWORD'),
	'doSomething' => mako\env('DO_SOMETHING', isBool: true),
]
```

You can also specify default values:

```
[
	'username'    => mako\env('USERNAME', 'user'),
	'password'    => mako\env('PASSWORD', '1234'),
	'doSomething' => mako\env('DO_SOMETHING', false, isBool: true),
]
```

--------------------------------------------------------

### <a id="package_configuration" href="#package_configuration">Package configuration</a>

Check out the [package documentation](:base_url:/docs/:version:/packages:packages#configuration_i18n_and_views) for more information regarding package configuration.
