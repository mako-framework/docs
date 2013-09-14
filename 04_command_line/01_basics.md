# CLI

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a CLI tool called ```reactor```. Reactor allows you to run parts of you application from the command line. This is especially useful when creating long running tasks such as cronjobs.

--------------------------------------------------------

<a id="usage"></a>

### Usage

Mako includes a several useful tasks by default. To list the available tasks just type this in your console:

	php reactor

It should print the following on a default Mako installation

	Usage:

	 php reactor <action> [arguments] [options]

	Global options:

	 --env                Allows you to override the default environment.
	 --database           Allows you to override the default database connection.

	Available actions:

	 app.up               Takes the application online.
	 app.down             Takes the application offline.
	 app.generate_secret  Generates a new application secret.

	 console              Starts a debug console.

	 migrate.status       Checks if there are any outstanding migrations.
	 migrate.create       Creates a new migration.
	 migrate.up           Runs all outstanding migrations.
	 migrate.down         Rolls back the last batch of migrations.
	 migrate.reset        Rolls back all migrations.

	 server               Starts a local development server.

Executing a task is done like this:

	php reactor app.down

You can list detailed information about a specific task like this:

	php reactor console --task-info

> All tasks have two special parameters, ```env``` and ```database``` which lets you override the environment and database connection to use.