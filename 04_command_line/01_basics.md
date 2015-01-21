# Command line

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a command line tool called ```reactor```. Reactor allows you to run parts of you application from the command line. This is especially useful when creating long running commands such as cronjobs.

--------------------------------------------------------

<a id="usage"></a>

### Usage

Mako includes a several useful commands by default. To list the available commands just type this in your console:

	php reactor

It should print the following on a default Mako installation

	╔═══╗
	║   ║ MAKO FRAMEWORK 4.4.0
	╚═══╝

	Usage:

	php reactor [command] [arguments] [options]

	Global options:

	----------------------------------------------------------
	| Option     | Description                               |
	----------------------------------------------------------
	| --mute     | Disables all output                       |
	| --env      | Overrides the Mako environment            |
	| --database | Overrides the default database connection |
	----------------------------------------------------------

	Available commands:

	-------------------------------------------------------------------------
	| Command             | Description                                     |
	-------------------------------------------------------------------------
	| app.generate_secret | Generates a new application secret.             |
	| app.routes          | Lists all registered routes.                    |
	| logo                |                                                 |
	| migrate.create      | Creates a new migration.                        |
	| migrate.down        | Rolls back the last batch of migrations.        |
	| migrate.reset       | Rolls back the last batch of migrations.        |
	| migrate.status      | Checks if there are any outstanding migrations. |
	| migrate.up          | Runs all outstanding migrations.                |
	| server              | Starts the local development server.            |
	-------------------------------------------------------------------------

Executing a commands is done like this:

	php reactor app.routes

You can list detailed information about a specific command like this:

	php reactor app.server --help