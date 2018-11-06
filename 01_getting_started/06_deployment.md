# Deployment

--------------------------------------------------------

* [Server configuration](#server_configuration)
	- [Nginx](#server_configuration:nginx)
* [Optimize Composer autoloading](#optimize_composer_autoloading)
* [Configure PHP for performance](#configure_php_for_performance)
	- [OPcache](#configure_php_for_performance:opcache)

--------------------------------------------------------

<a id="server_configuration"></a>

### Server configuration

<a id="server_configuration:nginx"></a>

#### Nginx

Basic [Nginx](http://nginx.org) configuration for a Mako application:

```
server
{
	listen 80;
	server_name example.org;
	root /srv/www/mako/htdocs/public;
	index index.php;

	access_log /srv/www/mako/logs/access.log;
	error_log  /srv/www/mako/logs/error.log;

	location /
	{
		try_files $uri $uri/ /index.php?$query_string;
	}

	location = /favicon.ico { access_log off; log_not_found off; }
	location = /robots.txt { access_log off; log_not_found off; }

	location ~* \.php$
	{
		try_files       $uri =404;
		include         fastcgi_params;
		fastcgi_pass    127.0.0.1:9000;
		fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
	}
}
```
{.language-nginx}

> Note that you can also use [php-fpm](https://php-fpm.org) over a unix socket instead of tcp. Just make sure that the [rlimit_files](http://php.net/manual/en/install.fpm.configuration.php) value is lower or equal to the file descriptor limit at the OS level.

--------------------------------------------------------

<a id="optimize_composer_autoloading"></a>

### Optimize Composer autoloading

You should consider running the following command as part of your deployment process as it will generate an optimized class autoloader.

```
composer dump-autoload --optimize --no-dev --classmap-authoritative
```
{.language-none}

--------------------------------------------------------

<a id="configure_php_for_performance"></a>

### Configure PHP for performance

<a id="configure_php_for_performance:opcache"></a>

#### OPcache

You should make sure that your production server has the [OPcache](http://php.net/manual/en/book.opcache.php) extension installed and enabled. OPcache will improve PHP performance by storing your application as compiled bytecode in shared memory, thereby removing the need for PHP to load and parse scripts on each request.

The following OPcache settings have been known to boost performance of a Mako application in production:

```
opcache.memory_consumption=256
opcache.interned_strings_buffer=16
opcache.max_accelerated_files=20000
opcache.validate_timestamps=0
opcache.fast_shutdown=1
```
{.language-none}

> Note that setting `validate_timestamps` to `0` tells OPcache to never check PHP files for changes. This is great for performance but it means that you'll have to clear the bytecode cache after each deployment to ensure that your files are recompiled.
>
> This can be done by reloading or restarting the php-fpm process, by calling `opcache_reset()` (this must be done via php-fpm) as part of your deployment or by using [cachetool](https://github.com/gordalina/cachetool).

You can also use [cachetool](https://github.com/gordalina/cachetool) to check the OPcache status. This can use useful if you want to see if you need to tweak some of the configuration values.
