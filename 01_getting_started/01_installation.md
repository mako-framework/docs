# Installation

--------------------------------------------------------

* [Requirements](#requirements)
* [Setup](#setup)
* [Server configurations](#server_configurations)

--------------------------------------------------------

<a id="requirements"></a>

### Requirements

* PHP 5.3.1 or higher (PHP 5.3.7 or higher is recommended)
* mbstring
* PDO

Mako has been tested on Apache, Cherokee, lighttpd and Nginx.

--------------------------------------------------------

<a id="setup"></a>

### Setup

Installing Mako is easy and can be with one single command thanks to [composer](https://packagist.org/):

	composer create-project mako/app <project name>

> Remember to make the ```app/storage/*``` directories writable.

Mako can now be updated using the following command:

	composer update

> You can also [download](:base_url:/download) a bare bones Mako application if you're unable to use composer.

--------------------------------------------------------

<a id="server_configurations"></a>

### Server configurations

#### Apache

Basic [Apache](http://www.apache.org/) configuration for a Mako application:

	<VirtualHost *:80>

		DocumentRoot /srv/www/mako/htdocs

		<Directory /srv/www/mako/htdocs>

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

#### Nginx

Basic [Nginx](http://nginx.org/) configuration for a Mako application:

	server
	{
		listen 80;

		access_log /srv/www/mako/logs/access.log;
		error_log  /srv/www/mako/logs/error.log;

		root /srv/www/mako/htdocs;

		index index.php;

		# Prevents access to the app and libraries directories

		location ~* ^/(app|libraries)
		{
			return 403;
		}

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