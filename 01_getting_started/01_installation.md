# Installation

--------------------------------------------------------

* [Requirements](#requirements)
* [Setup](#setup)
* [Updating](#updating)

--------------------------------------------------------

<a id="requirements"></a>

### Requirements

* PHP 7.0.0 or higher *
* mbstring
* PDO

\* Tested on PHP 7.0.x, 7.1.x and 7.2.x

--------------------------------------------------------

<a id="setup"></a>

### Setup

Installing Mako is easy and can be with done in a few simple steps thanks to [composer](https://packagist.org/).

First you'll have to create a new project:

```
composer create-project mako/app:5.* <project name>
```

Next you'll have to make the `app/storage/*` directories writable (command my vary depending on your system):

```
chown www-data:www-data -R app/storage
```

Now you're ready to start coding!

> Note that only the most essential [services](:base_url:/docs/:version:/getting-started:dependency-injection#services) are enabled by default. Enable the ones that you need by uncommenting them in the `app/config/application.php` configuration file.

--------------------------------------------------------

<a id="updating"></a>

### Updating

Mako can easily be updated when a new version is released using the following command:

```
composer update
```

> Note that some updates might require some minor code changes. These will be documented in the upgrade guides.
