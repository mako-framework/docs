# Command line

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Mako includes a command line tool called ```reactor```. Reactor comes with a set of useful commands out of the box and it allows you to create your own custom commands. This is especially useful when creating long running tasks such as cronjobs.

--------------------------------------------------------

<a id="usage"></a>

### Usage

To list the available commands just type this in your console:

	php reactor

It should print the following on a default Mako installation

	MAKO FRAMEWORK [ 5.0.0 ]

	Usage:

	php reactor [command] [arguments] [options]

	Global options:

	----------------------------------------------------------
	| Option     | Description                               |
	----------------------------------------------------------
	| --database | Overrides the default database connection |
	| --env      | Overrides the Mako environment            |
	| --mute     | Mutes all output                          |
	----------------------------------------------------------

	Available commands:

	-------------------------------------------------------------------------
	| Command             | Description                                     |
	-------------------------------------------------------------------------
	| app.generate_key    | Generates a 256-bit encryption key.             |
	| app.generate_secret | Generates a new application secret.             |
	| app.routes          | Lists all registered routes.                    |
	| migrate.create      | Creates a new migration.                        |
	| migrate.down        | Rolls back the last batch of migrations.        |
	| migrate.reset       | Rolls back the last batch of migrations.        |
	| migrate.status      | Checks if there are any outstanding migrations. |
	| migrate.up          | Runs all outstanding migrations.                |
	| server              | Starts the local development server.            |
	-------------------------------------------------------------------------

Executing a command is done like this:

	php reactor app.routes

You can list detailed information about a specific command like this:

	php reactor app.server --help