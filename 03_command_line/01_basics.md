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
MAKO FRAMEWORK [ 5.5.0 ]

Usage:

php reactor [command] [arguments] [options]

Global options:

-------------------------------------------
| Option | Description                    |
-------------------------------------------
| --env  | Overrides the Mako environment |
| --mute | Mutes all output               |
-------------------------------------------

Available commands:

-------------------------------------------------------------------------
| Command             | Description                                     |
-------------------------------------------------------------------------
| app.generate_key    | Generates a 256-bit encryption key.             |
| app.generate_secret | Generates a new application secret.             |
| app.routes          | Lists all registered routes.                    |
| cache.clear         | Clears the cache.                               |
| cache.remove        | Removes the chosen key from the cache.          |
| greeting            | Greets the user.                                |
| migrate.create      | Creates a new migration.                        |
| migrate.down        | Rolls back the last batch of migrations.        |
| migrate.reset       | Rolls back the last batch of migrations.        |
| migrate.status      | Checks if there are any outstanding migrations. |
| migrate.up          | Runs all outstanding migrations.                |
| server              | Starts the local development server.            |
-------------------------------------------------------------------------
```
{.language-none}

> Note that the list of available commands may vary depending on which services you have enabled.

Executing a command is easy and can be done like this:

```
php reactor app.routes
```
{.language-none}

You can list detailed information about a specific command using the `--help` flag.

```
php reactor app.server --help
```
{.language-none}
