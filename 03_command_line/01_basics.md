# Command line

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a command line tool called `reactor`. Reactor comes with a set of useful commands out of the box and you can also create your own custom commands. Commands are especially useful when creating long running tasks such as cron jobs or queue workers.

--------------------------------------------------------

<a id="usage"></a>

### Usage

To list the available commands just type the following in your console:

```
php reactor
```
{.language-none}

It should print something similar to the following on a default Mako installation.

```
MAKO FRAMEWORK [ 7.1.0 ]

Usage:

php reactor [command] [arguments] [options]

Global arguments and options:

-------------------------------------------------------
| Name    | Description                    | Optional |
-------------------------------------------------------
| command | Command name                   | Yes      |
| --env   | Overrides the Mako environment | Yes      |
| --help  | Displays helpful information   | Yes      |
| --mute  | Mutes all output               | Yes      |
-------------------------------------------------------

Available commands:

----------------------------------------------------------------------------
| Command                | Description                                     |
----------------------------------------------------------------------------
| app:generate-key       | Generates a 256-bit encryption key.             |
| app:generate-preloader | Generates a opcache preloader script.           |
| app:generate-secret    | Generates a new application secret.             |
| app:routes             | Lists all registered routes.                    |
| app:server             | Starts the local development server.            |
| cache:clear            | Clears the cache.                               |
| cache:remove           | Removes the chosen key from the cache.          |
| migration:create       | Creates a new migration.                        |
| migration:down         | Rolls back the last batch of migrations.        |
| migration:reset        | Resets the database schema.                     |
| migration:status       | Checks if there are any outstanding migrations. |
| migration:up           | Runs all outstanding migrations.                |
----------------------------------------------------------------------------
```
{.language-none}

> Note that the list of available commands may vary depending on which services you have enabled.

Executing a command is easy and can be done like this:

```
php reactor app:routes
```
{.language-none}

You can list detailed information about a specific command using the `--help` flag.

```
php reactor app:server --help
```
{.language-none}
