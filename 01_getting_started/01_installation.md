# Installation

--------------------------------------------------------

* [Requirements](#requirements)
* [Setup](#setup)
* [Server configurations](#server_configurations)
	- [Apache](#server_configurations:apache)
	- [Nginx](#server_configurations:nginx)

--------------------------------------------------------

<a id="requirements"></a>

### Requirements

* PHP 5.5.0 or higher *
* mbstring
* PDO

\* _Tested on PHP 5.5.x, 5.6.x and HHVM 3.x.x_

--------------------------------------------------------

<a id="setup"></a>

### Setup

Installing Mako is easy and can be with one single command thanks to [composer](https://packagist.org/):

	composer create-project mako/app:4.* <project name>

> Remember to make the ```app/storage/*``` directories writable.

Mako can now be updated using the following command:

	composer update

> You can also [download](:base_url:/download) a bare bones Mako application if you're unable to use composer.

--------------------------------------------------------

<a id="server_configurations"></a>

### Server configurations

<a id="server_configurations:apache"></a>

#### Apache

Basic [Apache](http://www.apache.org/) configuration for a Mako application:

	<VirtualHost *:80>

		DocumentRoot /srv/www/mako/htdocs/public

		<Directory /srv/www/mako/htdocs/public>

			Options -Indexes FollowSymLinks -MultiViews
			AllowOverride All
			Order allow,deny
			allow from all

			# URL rewrite

			RewriteEngine on

			RewriteCond %{REQUEST_FILENAME} !-f
			RewriteCond %{REQUEST_FILENAME} !-d
			RewriteRule ^(.*)$ index.php/$1 [L]

		</Directory>

		LogLevel warn
		ErrorLog /srv/www/mako/logs/error.log
		CustomLog /srv/www/mako/logs/access.log combined

	</VirtualHost>

<a id="server_configurations:nginx"></a>

#### Nginx

Basic [Nginx](http://nginx.org/) configuration for a Mako application:

	server
	{
		listen 80;

		access_log /srv/www/mako/logs/access.log;
		error_log  /srv/www/mako/logs/error.log;

		root /srv/www/mako/htdocs/public;

		index index.php;

		# "URL rewrite"

		location /
		{
			try_files $uri $uri/ /index.php?$query_string;
		}

		# Pass URIs ending in .php to the PHP interpreter

		location ~* \.php$
		{
			try_files       $uri =404;
			include         fastcgi_params;
			fastcgi_pass    127.0.0.1:9000;
			fastcgi_param   SCRIPT_FILENAME $document_root$fastcgi_script_name;
		}
	}