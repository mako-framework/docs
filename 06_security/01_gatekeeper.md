# Gatekeeper

--------------------------------------------------------

* [Users & Groups](#users_and_groups)
	- [Users](#users_and_groups:users)
		- [Password hashing](#users_and_groups:users:password_hashing)
		- [User repository](#users_and_groups:users:user_repository)
	- [Groups](#users_and_groups:groups)
		- [Group repository](#users_and_groups:groups:group_repository)
* [Authentication](#authentication)
* [Authorization](#authorization)
	- [Policies](#authorization:policies)
	- [Authorizing](#authorization:authorizing)
* [Database schema](#database_schema)
	- [MySQL](#database_schema:mysql)
	- [PostgreSQL](#database_schema:postgresql)
	- [SQLite](#database_schema:sqlite)
* [Adapters](#adapters)

--------------------------------------------------------

The gatekeeper library makes it easy to implement both authentication and authorization in your applications.

--------------------------------------------------------

<a id="users_and_groups"></a>

### Users and groups

We'll need users and maybe a group or two before we can start authenticating and authorizing so we'll start with explaining how to create and manage them.

<a id="users_and_groups:users"></a>

#### Users

The `createUser` method allows you to create a new user. A user object is returned upon successful creation.

```
$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password');

// You can also choose to activate the user upon creation

$user = $this->gatekeeper->createUser('foo@example.org', 'username', 'password', true);
```

> The gatekeeper user objects are [ORM models](:base_url:/docs/:version:/databases-sql:orm) that come with a [many to many](:base_url:/docs/:version:/databases-sql:orm#relations:many_to_many) relation to the group model and the group model has a many to many relation back to the user model.

If you chose to create a user without activating it then you'll probably want to implement email validation using the action token which can be retrieved from the user object using the `getActionToken` method.

```
$token = $user->getActionToken();

// Send the user an email with an activation link containing the token

...

```

To activate the user you'll have to use the `activateUser` method. The method will return `true` on success and `false` if the activation fails. The method will also automatically generate a new action token for the user upon successful activation.

```
$activated = $this->gatekeeper->activateUser($token);
```

Action tokens can also be used to validate "forgot password" requests (etc...) and should always be regenerated upon a successful action. This can be done using the `generateActionToken` method.

```
$user->generateActionToken();

$user->save();
```

Users have a second type of token called "access tokens" that are used by gatekeeper to identify users. The token can be retrieved using the `getAccessToken` method. New access token can be generated using the `generateAccessToken` method.

```
$user->generateAccessToken();

$user->save();
```

> Note that generating a new access token will invalidate all active sessions and "remember me" cookies for the user in question. You can use the gatekeeper [`forceLogin`](#authentication) method to log the user back in in the background to keep the experience seamless.

The user class also comes with the following methods in addition to the ones shown above:

| Method                                        | Description                                                                                   |
|-----------------------------------------------|-----------------------------------------------------------------------------------------------|
| getId()                                       | Returns the user id                                                                           |
| getEmail()                                    | Returns the email address                                                                     |
| setEmail($email)                              | Sets the email address                                                                        |
| getUsername()                                 | Returns the username                                                                          |
| setUsername($username)                        | Sets the username                                                                             |
| getPassword()                                 | Returns the password hash                                                                     |
| setPassword($password)                        | Sets the password and hashes it                                                               |
| getIp                                         | Returns the IP that the user had when registering                                             |
| setIp($ip)                                    | Sets the user ip                                                                              |
| activate()                                    | Activates the user                                                                            |
| deactivate()                                  | Deactivates the user                                                                          |
| isActivated()                                 | Returns `true` if the user is activated and `false` if not                                    |
| ban()                                         | Bans the user                                                                                 |
| unban()                                       | Unbans the user                                                                               |
| isBanned()                                    | Returns `true` if a user is banned and `false` if not                                         |
| isMemberOf($group)                            | Returns `true` if the user is a member of the group and `false` if not [†](#isMemberOf)       |
| validatePassword($password, $autosave = true) | Returns `true` if the provided password is correct and `false` if not [††](#validatePassword) |
| save()                                        | Saves the user state                                                                          |
| delete()                                      | Deletes the user                                                                              |

<a id="validatePassword"></a>
† `$group` can be a group name, a group id or an array of group names or ids. \
<a id="validatePassword"></a>
†† The method will also automatically rehash the password if necessary.

> Note that all methods that change the sate of the user require you to call the `save` method to persist the changes.

<a id="users_and_groups:users:password_hashing"></a>

##### Password hashing

User passwords are hashed using the [hashing library](:base_url:/docs/:version:/security:password-hashing). You can change the hashing algorithm or the default computing cost by reimplementing the `getHasher` method of the user class.

```
protected function getHasher(): HasherInterface
{
	return new Bcrypt(['cost' => 14]);
}
```

> Passwords will automatically be rehashed using the new algorithm or computing cost upon successful login and when successfuly validated using the `validatePassword` method.

<a id="users_and_groups:users:user_repository"></a>

##### User repository

Users can be fetched from the database using the user repository.

```
$userRepository = $this->gatekeeper->getUserRepository();
```

The user repository class comes with the following methods:

| Method                   | Description                                                                    |
|--------------------------|--------------------------------------------------------------------------------|
| getByActionToken($token) | Returns the user associated with the token or `null` if none is found         |
| getByAccessToken($token) | Returns the user associated with the token or `null` if none is found         |
| getByEmail($email)       | Returns the user associated with the email address or `null` if none is found |
| getById($id)             | Returns the user associated with the id or `null` if none is found            |

<a id="users_and_groups:groups"></a>

#### Groups

The `createGroup` method allows you to create a new group. A group object is returned upon successful creation.

```
$group = $this->gatekeeper->createGroup('admin');
```

> The gatekeeper group objects are [ORM models](:base_url:/docs/:version:/databases-sql:orm) that come with a [many to many](:base_url:/docs/:version:/databases-sql:orm#relations:many_to_many) relation to the user model and the user model has a many to many relation back to the group model.

The group class comes with the following methods:

| Method            | Description                                                            |
|-------------------|------------------------------------------------------------------------|
| getId()           | Returns the group id                                                   |
| getName()         | Returns the group name                                                 |
| setName($name)    | Sets the group name                                                    |
| addUser($user)    | Adds a user to the group                                               |
| removeUser($user) | Removes a user from the group                                          |
| isMember($user)   | Returns `true` if the user is a member of the group and `false` if not |
| save()            | Saves the group state                                                  |
| delete()          | Deletes the group                                                      |

<a id="users_and_groups:groups:group_repository"></a>

##### Group repository

Groups can be fetched from the database using the group repository.

```
$groupRepository = $this->gatekeeper->getGroupRepository();
```

The group repository class comes with the following methods:

| Method           | Description                                                             |
|------------------|-------------------------------------------------------------------------|
| getById($id)     | Returns the group associated with the id or `null` if none is found    |
| getByName($name) | Returns the group associated with the name or `null` if none is found  |

--------------------------------------------------------

<a id="authentication"></a>

### Authentication

The `login` method will attempt to log a user in. The method returns `true` if the login is successful and a status code if not.

```
$successful = $this->gatekeeper->login($email, $password);

// You can also tell gatekeeper to set a "remember me" cookie

$successful = $this->gatekeeper->login($email, $password, true);
```

The possible status codes for failed logins are:

| Constant                     | Description                                                                   |
|------------------------------|-------------------------------------------------------------------------------|
| Gatekeeper::LOGIN_INCORRECT  | The provided credentials are invalid                                          |
| Gatekeeper::LOGIN_ACTIVATING | The account has not been activated                                            |
| Gatekeeper::LOGIN_BANNED     | The account has been banned                                                   |
| Gatekeeper::LOGIN_LOCKED     | The account has been temporarily locked due to too many failed login attempts |

The `forceLogin` method allows you to login a user without a password. It will return `true` if the login is successful and a status code if not.

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

The `isGuest` method returns `false` if the user is logged in and `true` if not.

```
$isGuest = $this->gatekeeper->isGuest();
```

The `isLoggedIn` method returns `true` of the user is logged in and `false` if not.

```
$isLoggedIn = $this->gatekeeper->isLoggedIn();
```

The `getUser` method will return a user object if the user is logged in and `null` if not.

```
$user = $this->gatekeeper->getUser();
```

The `logout` method will log out the user and delete the "remember me" cookie if it is set.

```
$this->gatekeeper->logout();
```

--------------------------------------------------------

<a id="authorization"></a>

### Authorization

The authorization component of the gatekeeper library allows you to check if a user is allowed to perform a specific action on a entity.

<a id="authorization:policies"></a>

#### Policies

Authorization logic is defined in policy classes so that it can be reused anywhere in your application. In the example below we'll create a simple policy that authorizes the `view`, `create`, and `edit` actions on a article entity.

```
<?php

namespace app\policies;

use mako\gatekeeper\authorization\policies\Policy;
use mako\gatekeeper\entities\user\UserEntityInterface;

class ArticlePolicy extends Policy
{
	public function view(?UserEntityInterface $user, $entity): bool
	{
		return true;	
	}

	public function create(?UserEntityInterface $user, $entity): bool
	{
		if($user !== null && $user->isMemberOf('editors'))
		{
			return true;
		}

		return false;
	}

	public function edit(?UserEntityInterface $user, $entity): bool
	{
		if($user !== null && $user->getId() === $entity->user_id)
		{
			return true;
		}

		return false;
	}
}
```

The policy above extends the `Policy` class but you can choose to implement the `PolicyInterface` interface yourself if you want to implement the `before` method which lets you perform common checks in a single place.

```
public function before(?UserEntityInterface $user, string $action, $entity): ?bool
{
	if($user !== null && $user->isMemberOf('admins'))
	{
		return true;
	}

	return null;
}
```

> Returning a boolean value will prevent further authorization checks while returning `null` ensures normal authorization.

Before we can use the policy we'll have to associate it with the corresponding entity. This is done in the `app/config/gatekeeper.php` configuration file. The array key is the class name of the entity and the value is the class name of the corresponding policy.

```
[
	Article::class => ArticlePolicy::class,
]
```

<a id="authorization:authorizing"></a>

#### Authorizing

There are multiple ways of checking if a user is authorized to perform a certain action. The authorizer `can` method lets you authorize both guests and authenticated users.

```
if($this->authorizer->can($user, 'view', $article) === false)
{
	throw new ForbiddenException;
}

...
```

If you already know that you have an authenticated user then you can use the `can` method of the user class.

```
if($user->can('edit', $article) === false)
{
	throw new ForbiddenException;
}

...
```

And finally if your controller uses the `AuthorizationTrait` trait then you can use the `authorize` method that allows you to authorize both guests and authenticated users. It will automatically throw a `ForbiddenException` if the authorization fails.

```
$this->authorize('view', $article);

...
```

You'll not always be able to authorize actions on a entity instance so it also possible to pass the class name.

```
$this->authorize('create', Article::class);

...
```

--------------------------------------------------------

<a id="database_schema"></a>

### Database schema

The gatekeeper library requires three database tables: a users table, a groups table, and a junction table that links the two of them together. Here are the schemas for MySQL, PostgreSQL and SQLite.

<a id="database_schema:mysql"></a>

#### MySQL

[collapse:Click to view]
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
[/collapse]

<a id="database_schema:postgresql"></a>

#### PostgreSQL

[collapse:Click to view]
Users table

```
CREATE TABLE "users" (
	"id" SERIAL NOT NULL PRIMARY KEY,
	"created_at" TIMESTAMP(0) WITHOUT TIME ZONE NOT NULL,
	"updated_at" TIMESTAMP(0) WITHOUT TIME ZONE NOT NULL,
	"ip" VARCHAR(255) NOT NULL,
	"username" VARCHAR(255) NOT NULL UNIQUE,
	"email" VARCHAR(255) NOT NULL UNIQUE,
	"password" VARCHAR(255) NOT NULL,
	"action_token" CHAR(64) DEFAULT '',
	"access_token" CHAR(64) DEFAULT '',
	"activated" BOOLEAN NOT NULL DEFAULT FALSE,
	"banned" BOOLEAN NOT NULL DEFAULT FALSE,
	"failed_attempts" INTEGER NOT NULL DEFAULT 0,
	"last_fail_at" TIMESTAMP(0) WITHOUT TIME ZONE DEFAULT NULL,
	"locked_until" TIMESTAMP(0) WITHOUT TIME ZONE DEFAULT NULL
);
```
{.language-sql}

Groups table

```
CREATE TABLE "groups" (
	"id" SERIAL NOT NULL PRIMARY KEY,
	"created_at" TIMESTAMP(0) WITHOUT TIME ZONE NOT NULL,
	"updated_at" TIMESTAMP(0) WITHOUT TIME ZONE NOT NULL,
	"name" VARCHAR(255) NOT NULL UNIQUE
);
```
{.language-sql}

Junction table

```
CREATE TABLE "groups_users" (
	"group_id" INTEGER NOT NULL REFERENCES "groups" ON DELETE CASCADE,
	"user_id" INTEGER NOT NULL REFERENCES "users" ON DELETE CASCADE,
	UNIQUE ("group_id", "user_id")
);

CREATE INDEX "groups_users_group_id_idx" ON "groups_users" USING btree("group_id");

CREATE INDEX "groups_users_user_id_idx" ON "groups_users" USING btree("user_id");
```
{.language-sql}
[/collapse]

<a id="database_schema:sqlite"></a>

#### SQLite

[collapse:Click to view]
Users table

```
CREATE TABLE `users` (
	`id` INTEGER PRIMARY KEY AUTOINCREMENT,
	`created_at` TEXT NOT NULL,
	`updated_at` TEXT NOT NULL,
	`ip` TEXT(255) NOT NULL,
	`username` TEXT(255) NOT NULL,
	`email` TEXT(255) NOT NULL,
	`password` TEXT(255) NOT NULL,
	`action_token` TEXT(64) DEFAULT '',
	`access_token` TEXT(64) DEFAULT '',
	`activated` TINYINT NOT NULL DEFAULT 0,
	`banned` TINYINT NOT NULL DEFAULT 0,
	`failed_attempts` INTEGER NOT NULL DEFAULT 0,
	`last_fail_at` TEXT DEFAULT NULL,
	`locked_until` TEXT DEFAULT NULL,
	CONSTRAINT `username` UNIQUE (`username`),
	CONSTRAINT `email` UNIQUE (`email`)
);
```
{.language-sql}

Groups table

```
CREATE TABLE `groups` (
	`id` INTEGER PRIMARY KEY AUTOINCREMENT,
	`created_at` TEXT NOT NULL,
	`updated_at` TEXT NOT NULL,
	`name` TEXT(255) NOT NULL,
	CONSTRAINT `name` UNIQUE (`name`)
);
```
{.language-sql}

Junction table

```
CREATE TABLE `groups_users` (
	`group_id` INTEGER NOT NULL,
	`user_id` INTEGER NOT NULL,
	CONSTRAINT `group_user` UNIQUE (`group_id`, `user_id`),
	FOREIGN KEY (`group_id`)
		REFERENCES `groups` (`id`)
		ON DELETE CASCADE ON UPDATE NO ACTION,
	FOREIGN KEY (`user_id`)
		REFERENCES `users` (`id`)
		ON DELETE CASCADE ON UPDATE NO ACTION
);
CREATE INDEX `group_id` ON `groups_users` (`group_id`);
CREATE INDEX `user_id` ON `groups_users` (`user_id`);
```
{.language-sql}
[/collapse]

--------------------------------------------------------

<a id="adapters"></a>

### Adapters

Gatekeeper comes with a session based adapter out of the box but you can implement your own custom adapters as well.

In the examples above we have chosen to use the gatekeeper class as a proxy to the default adapter. You can specify which adapter to use by calling the `adapter` method.

```
// Default adapter

$adapter = $this->gatekeeper->adapter();

// Custom adapter

$adapter = $this->gatekeeper->adapter('custom');
```

You can choose to replace the default adapter by creating a custom [`GatekeeperService`](:base_url:/docs/:version:/getting-started:dependency-injection#services) or you can add additional adapters by registering them using the `extend` method.

```
// Register an instance

$this->gatekeeper->extend(new CustomAdapter);

// Register an adapter factory where the first array element
// is the adapter name while the second is the factory

$this->gatekeeper->extend(['custom', fn () => new CustomAdapter])
```

> Note that all adapters must implement the `AdapterInterface`.

You can switch the default adapter at any time by using the `useAsDefaultAdapter` method.

```
$this->gatekeeper->useAsDefaultAdapter('custom');
````
