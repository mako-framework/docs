# REPL

--------------------------------------------------------

* [Installation](#installation)
* [Usage](#usage)

--------------------------------------------------------

The Mako REPL package is a runtime developer console, interactive debugger and [REPL](https://en.wikipedia.org/wiki/Read-eval-print_loop) built on top of the awesome [Psy Shell](https://psysh.org).

--------------------------------------------------------

<a id="installation"></a>

### Installation

Install the package using the following composer command:

```
composer require mako/repl
```
{.language-none}

Next, add the `mako\repl\ReplPackage` package to your `app/config/application.php` config file.

--------------------------------------------------------

<a id="usage"></a>

### Usage

Start the interactive shell using the following reactor command.

```
php app/reactor repl
```
{.language-none}

You should see something similar to this:

```
----------------------------------------------
Type help to see a list of available commands.
----------------------------------------------
Psy Shell v0.8.0 (PHP 7.0.14 — cli) by Justin Hileman
>>>
```
{.language-none}

There's an anonymous container aware class instance available as `$mako` by default. This allows you to easily interact with services registered in the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection#services).

The object also has a public method called `getContainer()` which, as the name suggests, allows you to access the container directly.

In the example below we're selecting all records from our `tests` column using the query builder.

```
>>> $mako->database->table('tests')->all()
=> mako\database\query\ResultSet {#229
	+0: mako\database\query\Result {#230
		+id: 1,
		+created_at: "2016-12-30 10:58:56",
	},
	+1: mako\database\query\Result {#231
		+id: 2,
		+created_at: "2016-12-31 01:21:20",
	},
}
```
{.language-none}
