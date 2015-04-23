# Upgrading

--------------------------------------------------------

* [Application](#application)
	- [Configuration](#application:configuration)
	- [Structure](#application:structure)
* [Packages](#packages)
	- [Structure](#packages:structure)
* [Database](#database)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```4.4.x``` to ```4.5.x```.

--------------------------------------------------------

<a id="application"></a>

### Application

<a id="application:configuration"></a>

#### Configuration

The ```app/config/gatekeeper.php``` configuration file has been updated with a set of new brute force throttling [options](https://github.com/mako-framework/app/blob/8df4487ac1e3534a3225c79723749169f5f528ad/app/config/gatekeeper.php#L48).

<a id="application:structure"></a>

#### Structure

There has been made some minor changes to the application structure:

* The ```app/commands``` directory has been moved to ```app\console\commands```.
* The ```app/views``` directory has been moved to ```app\resources\views```.
* The ```app/i18n``` directory has been moved to ```app\resources\i18n```.

--------------------------------------------------------

<a id="packages"></a>

### Packages

<a id="packages:structure"></a>

#### Structure

There has been made some minor changes to the default package structure.

* The ```views``` directory has been moved to ```resources\views```.
* The ```i18n``` directory has been moved to ```resources\i18n```.

--------------------------------------------------------

<a id="database"></a>

### Database

Three new fields have been added to the [gatekeeper users table](:base_url:/docs/:version:/security:authentication#database_schema) (```users.failed_attempts```, ```users.last_fail_at``` and ```users.locked_until```).