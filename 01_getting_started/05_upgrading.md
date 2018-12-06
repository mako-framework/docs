# Upgrading

--------------------------------------------------------

* [Framework](#framework)
	- [security](#framework:security)
		- [Gatekeeper](#framework:security:gatekeeper)
		- [Password hashing](#framework:security:password_hashing)
	- [Database](#framework:database)
		- [Query builder](#framework:database:query_builder)
		- [ORM](#framework:database:orm)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `5.6.x` to `5.7.x`.

--------------------------------------------------------

### Framework

<a id="framework"></a>

#### Security

<a id="framework:security"></a>

##### Gatekeeper

<a id="framework:security:gatekeeper"></a>

The `forceLogin` method will now return `true` when a login is successful and a status code instead of `false` when the login fails.

```
// Before

if($gatekeeper->forceLogin($identifier) === false)
{

}

// Now

if($gatekeeper->forceLogin($identifier) !== true)
{

}
```

##### Password hashing

<a id="framework:security:password_hashing"></a>

The `mako\security\Password` class has been removed. If you were using it they you'll have to update your code to use the new [password hashing library](:base_url:/docs/:version:/security:password-hashing) introduced in `5.6.0`.

#### Database

<a id="framework:database"></a>

##### Query builder

<a id="framework:database:query_builder"></a>

The JSON representation of a result set returned by the `Query::paginate()` method will now be an object where the results are available as data and pagination information will be available as pagination. See the [query builder documentation](:base_url:/docs/:version:/databases-sql:query-builder#array_and_json_representations) for more details.

##### ORM

<a id="framework:database:orm"></a>

The `ORM::$exists` property and `ORM::exists()` method has been removed. Use `ORM::$isPersisted` and `ORM::isPersisted()` instead.
