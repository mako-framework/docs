# Migrations

--------------------------------------------------------

* [Enabling migrations](#enabling_migrations)
* [Creating migrations](#creating_migrations)
* [Running migrations](#running_migrations)
* [Rolling back migrations](#rolling_back_migrations)

--------------------------------------------------------

Database migrations offer a convenient way to alter your databases in a structured and organized manner. They allow you to commit and roll back schema changes.

Mako migrations are created and executed from the [reactor CLI tool](:base_url:/docs/:version:/cli:tasks).

--------------------------------------------------------

<a id="enabling_migrations"></a>

### Enabling migrations

You need to add a table called ```mako_migrations``` to your database. The table will be used to keep track of the migrations that have been ran. Here's the SQL for creating the table using MySQL:

	CREATE TABLE `mako_migrations`
	(
		`batch` int(10) unsigned NOT NULL,
		`package` varchar(255) NOT NULL,
		`version` varchar(255) NOT NULL
	);

--------------------------------------------------------

<a id="creating_migrations"></a>

### Creating migrations

Creating a migration is done from the reactor CLI tool:

	php reactor migrate.create        // Creates an application migration
	php reactor migrate.create foobar // Creates a migration for the "foobar" package

Running the ```create``` commands will return the following messages:

	Migration created at "/var/www/app/migrations/20120824100019.php".
	Migration created at "/var/www/app/packages/foobar/migrations/20120824100317.php".

The generated migration will contain a skeleton class with two methods, ```up``` and ```down```.

	class Migration_20120824100019
	{
	        /**
	        * Makes changes to the database structure.
	        *
	        * @access  public
	        */

	        public function up()
	        {

	        }

	        /**
	        * Reverts the database changes.
	        *
	        * @access  public
	        */

	        public function down()
	        {

	        }
	}

--------------------------------------------------------

<a id="running_migrations"></a>

### Running migrations

You can check if there are any outstanding migrations using the ```status``` command:

	php reactor migrate.status

If there are outstanding migrations then you can run them like this:

	php reactor migrate.up

This will show you the names of the migrations that were executed:

	Ran the 20120824100019 migration.
	Ran the foobar::20120824100317 migration.

--------------------------------------------------------

<a id="rolling_back_migrations"></a>

### Rolling back migrations

If you need to revert the changes made to your database then you can use the ```down``` command. This will roll back the last batch of migrations executed.

	php reactor migrate.down

This will show you the migrations that were rolled back:

	Rolled back the foobar::20120824100317 migration.
	Rolled back the 20120824100019 migration.

You can roll back multiple batches by telling the rollback command how many batches you want to roll back.

	php reactor migrate.down 2

If you want to roll back all database changes in one go then you can use the ```reset``` command.

	php reactor migrate.reset

This will prompt you for confirmation. To force the reset just use the ```force``` option.

	php reactor migrate.reset --force