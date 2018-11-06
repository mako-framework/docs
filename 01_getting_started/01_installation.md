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

\* Tested on PHP 7.0.x, 7.1.x, 7.2.x and 7.3.x

--------------------------------------------------------

<a id="setup"></a>

### Setup

Installing Mako is easy and can be with done in a few simple steps thanks to [composer](https://packagist.org/).

First you'll have to create a new project:

```
composer create-project mako/app <project name>
```
{.language-none}

Next you'll have to make the `app/storage/*` directories writable (command my vary depending on your system):

```
chown www-data:www-data -R app/storage
```
{.language-none}

You can now start a simple development server with the following command:

```
php app/reactor server
```
{.language-none}

Open `http://localhost:8000` in your browser of choice and you're ready to start coding!

> Note that only the most essential [services](:base_url:/docs/:version:/getting-started:dependency-injection#services) are enabled by default. Enable the ones that you need by uncommenting them in the `app/config/application.php` configuration file.

--------------------------------------------------------

<a id="updating"></a>

### Updating

Mako can easily be updated when a new patch release is made available using the following command:

```
composer update
```
{.language-none}

> If you want to bump the version (e.g. from `5.6.*` to `5.7.*`) then you'll have to update your `composer.json` file before running the update command. Note that some releases might require some minor code changes. These will be documented in the upgrade guides.
