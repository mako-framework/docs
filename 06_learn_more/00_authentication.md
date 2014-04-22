# Authentication

--------------------------------------------------------

* [Basic usage](#basic_usage)
* [Users & groups](#users_and_groups)
	- [Basics](#users_and_groups_basics)
	- [Passwords](#users_and_groups_passwords)
* [Database schema](#database_schema)

--------------------------------------------------------

The gatekeeper library allows you to easily implement user authentication in your application.

--------------------------------------------------------

<a id="basic_usage"></a>

### Basic usage

The ```createUser``` method allows you to create a new user. A user object is returned upon successful creation.

	$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password');

	// You can also choose to activate the user upon creation

	$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password', true);

The ```activateUser``` method activates a user. It will return TRUE on success and FALSE if the activation fails.

	$activated = $this->gatekeeper->activateUser($token);

The ```login``` method will attempt to log a user in. The method returns TRUE if the login is successfull and a status code if not. 

The possible status codes for failed logins are ```Gatekeeper::LOGIN_ACTIVATING```, ```Gatekeeper::LOGIN_BANNED``` and ```Gatekeeper::LOGIN_INCORRECT```.

	$successful = $this->gatekeeper->login($email, $password);

	// You can also tell gatekeeper to set a "remember me" cookie

	$successful = $this->gatekeeper->login($email, $password, true);

The ```forceLogin``` allows you to login a user without a password. It will return TRUE if the login is successful and one a status code if not. 

The possible status codes for failed logins are ```Gatekeeper::LOGIN_ACTIVATING``` and ```Gatekeeper::LOGIN_BANNED```.

	$successful = $this->gatekeeper->forceLogin($email);

	// You can also tell gatekeeper to set a "remember me" cookie

	$successful = $this->gatekeeper->forceLogin($email, true);

The ```basicAuth``` method can be usefull when creating APIs or if you don't want to create a full login page. It will prompt for login if a valid session isn't present.

	$this->gatekeeper->basicAuth();

> The username and password is sent with every subsequent request so use HTTPS if possible!

The ```isGuest``` method returns FALSE if the user is logged in and TRUE if not.

	$isGuest = $this->gatekeeper->isGuest();

The ```isLoggedIn``` method returns TRUE of the user is logged in and FALSE if not.

	$isLoggedIn = $this->gatekeeper->isLoggedIn();

The ```user``` method will return a user object if the user is logged in and NULL if not.

	$user = $this->gatekeeper->user();

The ```logout()``` method will log out the user and delete the "remember me" cookie if it is set.

	$this->gatekeeper->logout();

--------------------------------------------------------

<a id="users_and_groups"></a>

### Users & groups

<a id="users_and_groups_basics"></a>

#### Basics

The user object returned by gatekeeper is a [ORM model](:base_url:/docs/:version:/databases:orm). The model (```mako\auth\models\User```) comes with a few extra methods and a [many to many](:base_url:/docs/:version:/databases:orm#relations:many_to_many) relation to the ```mako\auth\models\Group``` model. The group model has a many to many relation back to the user model.

The ```validatePassword``` method allows you validate a user password.

	$user = $this->gatekeeper->user();

	$isValid = $user->validatePassword($password);

The ```generateToken``` method allows you to generate a new authentication token for the user.

	$user->generateToken();

	$user->save();

The ```memberOf``` method allows you to check whether or not a user is a member of a group or a set of groups.

	$isMemberOf = $user->memberOf('admin');

	// Returns true if the user is a member "staff" or "admin"

	$isMemberOf = $user->memberOf(['staff', 'admin']);

The ```isActivated``` method returns TRUE if the user is activated and FALSE if not.

	$activated = $user->isActivated();

The ```activate``` method activates a user.

	$user->activate();

	$user->save();

The ```deactivate``` method will deactivate a user.

	$user->deactivate();

	$user->save();

The ```isBanned``` method will return TRUE if a user is banned and FALSE if not.

	$isBanned = $user->isBanned();

The ```ban``` method will ban a user.

	$user->ban();

	$user->save();

The ```unban``` method will unban a user.

	$user->unban();

	$user->save();

<a id="users_and_groups_passwords"></a>

#### Passwords

You set a user password using the ```$password``` property of the user object. The password will automatically be hashed using a [salted bcrypt hash](:base_url:/docs/:version:/learn-more:password-hashing) so you do not need to hash it yourself.

	$user->password = 'my new super secret password';

	$user->generateToken();

	$user->save();

> You should always generate a new token when a user updates his or her password! Remember to create a new session using the ```forceLogin``` method to prevent the user from getting logged out on the next pageload.

--------------------------------------------------------

<a id="database_schema"></a>

### Database schema

The authentication library requires three database tables. Here are the create statements for MySQL.

Users table

	CREATE TABLE `users` (
	  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
	  `ip` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	  `created_at` datetime NOT NULL,
	  `email` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	  `username` varchar(255) CHARACTER SET utf8 COLLATE utf8_unicode_ci NOT NULL DEFAULT '',
	  `password` char(60) COLLATE utf8_unicode_ci NOT NULL DEFAULT '',
	  `token` char(32) COLLATE utf8_unicode_ci NOT NULL DEFAULT '',
	  `activated` set('0','1') COLLATE utf8_unicode_ci NOT NULL DEFAULT '0',
	  `banned` set('0','1') COLLATE utf8_unicode_ci NOT NULL DEFAULT '0',
	  PRIMARY KEY (`id`),
	  UNIQUE KEY `username` (`username`),
	  UNIQUE KEY `email` (`email`)
	) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

Junction table

	CREATE TABLE `groups_users` (
	  `gatekeeperuser_id` int(11) unsigned NOT NULL,
	  `gatekeepergroup_id` int(11) unsigned NOT NULL,
	  UNIQUE KEY `user_id_2` (`gatekeeperuser_id`,`gatekeepergroup_id`),
	  KEY `user_id` (`gatekeeperuser_id`),
	  KEY `group_id` (`gatekeepergroup_id`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

Groups table

	CREATE TABLE `groups` (
	  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
	  `name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	  PRIMARY KEY (`id`),
	  UNIQUE KEY `name` (`name`)
	) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;