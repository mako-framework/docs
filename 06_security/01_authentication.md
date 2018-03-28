# Authentication

--------------------------------------------------------

* [Basic usage](#basic_usage)
* [Repositories](#repositories)
	- [User repository](#repositories_user)
	- [Group repository](#repositories_group)
* [Users & groups](#users_and_groups)
	- [Users](#users_and_groups_users)
	- [Groups](#users_and_groups_groups)
* [Database schema](#database_schema)
	- [MySQL](#database_schema:mysql)
	- [PostgreSQL](#database_schema:postgresql)

--------------------------------------------------------

The gatekeeper library allows you to easily implement user authentication in your application.

--------------------------------------------------------

<a id="basic_usage"></a>

### Basic usage

The `createUser` method allows you to create a new user. A user object is returned upon successful creation.

```
$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password');

// You can also choose to activate the user upon creation

$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password', true);
```

The `createGroup` method allows you to create a new group. A group object is returned upon successful creation.

```
$group = $this->gatekeeper->createGroup('admin');
```

The `activateUser` method activates a user by his or her action token. The method will return TRUE on success and FALSE if the activation fails. The method will also automatically generate a new action token for the user.

```
$activated = $this->gatekeeper->activateUser($token);
```

The `login` method will attempt to log a user in. The method returns TRUE if the login is successful and a status code if not.

```
$successful = $this->gatekeeper->login($email, $password);

// You can also tell gatekeeper to set a "remember me" cookie

$successful = $this->gatekeeper->login($email, $password, true);
```

The possible status codes for failed logins are:

| Constant                         | Description                                                                   |
|----------------------------------|-------------------------------------------------------------------------------|
| Authentication::LOGIN_INCORRECT  | The provided credentials are invalid                                          |
| Authentication::LOGIN_ACTIVATING | The account has not been activated                                            |
| Authentication::LOGIN_BANNED     | The account has been banned                                                   |
| Authentication::LOGIN_LOCKED     | The account has been temporarily locked due to too many failed login attempts |

The `forceLogin` method allows you to login a user without a password. It will return TRUE if the login is successful and a status code if not.

```
$successful = $this->gatekeeper->forceLogin($email);

// You can also tell gatekeeper to set a "remember me" cookie

$successful = $this->gatekeeper->forceLogin($email, true);
```

The `basicAuth` method can be useful when creating APIs or if you don't want to create a full login page. It will return `true` if the user is logged in and `false` if not.

```
if($this->gatekeeper->basicAuth() === false)
{
	return 'Authentication required.';
}

// Code here gets executed if the user is logged in
```

> The username and password is sent with every subsequent request when using basic authentication so make sure to use `HTTPS` whenever possible!
{.danger}

The `isGuest` method returns FALSE if the user is logged in and TRUE if not.

```
$isGuest = $this->gatekeeper->isGuest();
```

The `isLoggedIn` method returns TRUE of the user is logged in and FALSE if not.

```
$isLoggedIn = $this->gatekeeper->isLoggedIn();
```

The `getUser` method will return a user object if the user is logged in and NULL if not.

```
$user = $this->gatekeeper->getUser();
```

The `logout()` method will log out the user and delete the "remember me" cookie if it is set.

```
$this->gatekeeper->logout();
```

The `getUserRepository` method returns the gatekeeper [user repository](#repositories_user).

```
$userRepository = $this->gatekeeper->getUserRepository();
```

The `getGroupRepository` method returns the gatekeeper [group repository](#repositories_group).

```
$groupRepository = $this->gatekeeper->getGroupRepository();
```

--------------------------------------------------------

<a id="repositories_group"></a>

### Repositories

<a id="repositories_user"></a>

#### User repository

The `getByActionToken` method returns a user based on his or her action token and FALSE if no user is found.

```
$user = $userRepository->getByActionToken($token);
```

The `getByAccessToken` method returns a user based on his or her access token and FALSE if no user is found.

```
$user = $userRepository->getByAccessToken($token);
```

The `getByEmail` method returns a user based on his or her email address and FALSE if no user is found.

```
$user = $userRepository->getByEmail($email);
```

The `getById` method returns a user based on his or her id and FALSE if no user is found.

```
$user = $userRepository->getById($id);
```

<a id="repositories_group"></a>

#### Group repository

The `getByName` method returns a group based on its name and FALSE if no group is found.

```
$group = $groupRepository->getByName($name);
```

The `getById` method returns a group based on its id and FALSE if no group is found.

```
$group = $groupRepository->getById($id);
```

--------------------------------------------------------

<a id="users_and_groups"></a>

### Users & groups

The user and group objects returned by the default gatekeeper implementation are [ORM models](:base_url:/docs/:version:/databases-sql:orm).

The user model comes with a [many to many](:base_url:/docs/:version:/databases-sql:orm#relations:many_to_many) relation to the group model and the group model has a many to many relation back to the user model.

<a id="users_and_groups_users"></a>

#### Users

The `generateAccessToken` method allows you to generate a new access token for the user.

```
$user->generateAccessToken();
```

> Generating a new token will invalidate all active sessions and "remember me" cookies for the user in question. You can use the `Authentication::forceLogin` method to log the user back in in the background to keep the experience seamless.

The `generateActionToken` method allows you to generate a new action token for the user. Should be used to activate a user, to validate "forgot password" requests etc.

```
$user->generateActionToken();
```

> You should generate a new action token after a successful action. Note that gatekeeper automatically generates a new action token when activating a user.
{.warning}

The `isMemberOf` method allows you to check whether or not a user is a member of a group or a set of groups.

```
$isMemberOf = $user->isMemberOf('admin');

// Returns true if the user is a member of "staff" or "admin"

$isMemberOf = $user->isMemberOf(['staff', 'admin']);
```

The `isActivated` method returns TRUE if the user is activated and FALSE if not.

```
$activated = $user->isActivated();
```

The `activate` method activates a user.

```
$user->activate();
```

The `deactivate` method will deactivate a user.

```
$user->deactivate();
```

The `isBanned` method will return TRUE if a user is banned and FALSE if not.

```
$isBanned = $user->isBanned();
```

The `ban` method will ban a user.

```
$user->ban();
```

The `unban` method will unban a user.

```
$user->unban();
```

The `save` method allows you to save changes to the user.

```
$user->save();
```

The `delete` method allows you to delete a user.

```
$user->delete();
```

The user object also includes the following getters and setters: `getId`, `setEmail`, `getEmail`, `setUsername`, `getUsername`, `setPassword`, `getPassword`, `setIp`, `getIp`, `getActionToken` and `getAccessToken`.

> The password will automatically be hashed using a [salted bcrypt hash](:base_url:/docs/:version:/security:password-hashing) so you do not need to hash it yourself.

<a id="users_and_groups_groups"></a>

#### Groups

The `addUser` method adds a user to the group.

```
$group->addUser($user);
```

The `removeUser` method removes a user from the group.

```
$group->removeUser($user);
```

The `isMember` method returns TRUE if the member is a member of the group and FALSE if not.

```
$group->isMember($user);
```

The `save` method allows you to save changes to the group.

```
$group->save();
```

The `delete` method allows you to delete a group.

```
$group->delete();
```

The group object also includes the following getters and setters: `getId`, `setName` and `getName`.

--------------------------------------------------------

<a id="database_schema"></a>

### Database schema

The authentication library requires three database tables: a users table, a groups table, and a junction table combining the two. Here are the schemas for MySQL and PostgreSQL.

<a id="database_schema:mysql"></a>

#### MySQL

Users table

```
CREATE TABLE `users` (
	`id` int(11) unsigned NOT NULL AUTO_INCREMENT,
	`created_at` datetime NOT NULL,
	`updated_at` datetime NOT NULL,
	`ip` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	`username` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	`email` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	`password` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	`action_token` char(64) COLLATE utf8_unicode_ci DEFAULT '',
	`access_token` char(64) COLLATE utf8_unicode_ci DEFAULT '',
	`activated` tinyint(1) NOT NULL DEFAULT 0,
	`banned` tinyint(1) NOT NULL DEFAULT 0,
	`failed_attempts` int(11) NOT NULL DEFAULT '0',
	`last_fail_at` datetime DEFAULT NULL,
	`locked_until` datetime DEFAULT NULL,
	PRIMARY KEY (`id`),
	UNIQUE KEY `username` (`username`),
	UNIQUE KEY `email` (`email`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
{.language-sql}

Groups table

```
CREATE TABLE `groups` (
	`id` int(11) unsigned NOT NULL AUTO_INCREMENT,
	`created_at` datetime NOT NULL,
	`updated_at` datetime NOT NULL,
	`name` varchar(255) COLLATE utf8_unicode_ci NOT NULL,
	PRIMARY KEY (`id`),
	UNIQUE KEY `name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
{.language-sql}

Junction table

```
CREATE TABLE `groups_users` (
	`group_id` int(11) unsigned NOT NULL,
	`user_id` int(11) unsigned NOT NULL,
	UNIQUE KEY `group_user` (`group_id`,`user_id`),
	KEY `group_id` (`group_id`),
	KEY `user_id` (`user_id`),
	CONSTRAINT `groups`
		FOREIGN KEY (`group_id`)
		REFERENCES `groups` (`id`)
		ON DELETE CASCADE ON UPDATE NO ACTION,
	CONSTRAINT `users`
		FOREIGN KEY (`user_id`)
		REFERENCES `users` (`id`)
		ON DELETE CASCADE ON UPDATE NO ACTION
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```
{.language-sql}

<a id="database_schema:postgresql"></a>

#### PostgreSQL

Users table

```
CREATE TABLE users (
	id SERIAL NOT NULL PRIMARY KEY,
	created_at TIMESTAMP NOT NULL,
	updated_at TIMESTAMP NOT NULL,
	ip VARCHAR(255) NOT NULL,
	username VARCHAR(255) NOT NULL UNIQUE,
	email VARCHAR(255) NOT NULL UNIQUE,
	password VARCHAR(255) NOT NULL,
	action_token CHAR(64) DEFAULT '',
	access_token CHAR(64) DEFAULT '',
	activated BOOLEAN NOT NULL DEFAULT FALSE,
	banned BOOLEAN NOT NULL DEFAULT FALSE,
	failed_attempts INTEGER NOT NULL DEFAULT 0,
	last_fail_at TIMESTAMP DEFAULT NULL,
	locked_until TIMESTAMP DEFAULT NULL
);
```
{.language-sql}

Groups table

```
CREATE TABLE groups (
	id SERIAL NOT NULL PRIMARY KEY,
	created_at TIMESTAMP NOT NULL,
	updated_at TIMESTAMP NOT NULL,
	name VARCHAR(255) NOT NULL UNIQUE
);
```
{.language-sql}

Junction table

```
CREATE TABLE groups_users (
	group_id INTEGER NOT NULL REFERENCES groups ON DELETE CASCADE,
	user_id INTEGER NOT NULL REFERENCES users ON DELETE CASCADE,
	PRIMARY KEY(group_id, user_id)
);
```
{.language-sql}
