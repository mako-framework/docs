# CLI

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a command line tool called ```reactor```. Reactor allows you to run parts of you application from the command line. This is especially useful when creating long running tasks such as cronjobs.

--------------------------------------------------------

<a id="usage"></a>

### Usage

Mako includes a several useful tasks by default. To list the available tasks just type this in your console:

	php reactor

It should print the following on a default Mako installation

	Usage:

	php reactor <action> [arguments] [options]

	Global options:

	+------------+---------------------------------------------------------+
	| Option     | Description                                             |
	+------------+---------------------------------------------------------+
	| --env      | Allows you to override the default environment.         |
	| --database | Allows you to override the default database connection. |
	| --hush     | Disables all output                                     |
	+------------+---------------------------------------------------------+

	Available actions:

	+---------------------------+-------------------------------------------------+
	| Action                    | Description                                     |
	+---------------------------+-------------------------------------------------+
	| mako::app.generate_secret | Generates a new application secret.             |
	| mako::app.routes          | Lists the registered routes of the application. |
	| mako::console.start       | Starts the debug console.                       |
	| mako::migrate.status      | Checks if there are any outstanding migrations. |
	| mako::migrate.create      | Creates a new migration.                        |
	| mako::migrate.up          | Runs all outstanding migrations.                |
	| mako::migrate.down        | Rolls back the last batch of migrations.        |
	| mako::migrate.reset       | Rolls back all migrations.                      |
	| mako::server.start        | Starts the local development server.            |
	+---------------------------+-------------------------------------------------+

Executing a task is done like this:

	php reactor mako::app.routes

You can list detailed information about a specific task like this:

	php reactor mako::app --task-info

> All tasks have three special options, ```env``` (lets you override the environment), ```database``` (lets your set the default database connection) and ```hush``` (disables all output).