# Password hashing

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Using md5 or sha1 hashes for storing passwords is not recommended, as they are easy to brute-force with modern hardware. The password hashing class makes it easy to hash and validate your passwords using a secure method.

The password hashing class uses PHP's `password_*` methods internally. Passwords are currently hashed using the `PASSWORD_BCRYPT` algorithm (default as of PHP 5.5.0) but this may change over time as new and stronger algorithms are added to PHP.

All examples where we override the computing cost assume that `PASSWORD_BCRYPT` is the default hashing algorithm on your system. See the [PHP documentation](http://php.net/manual/en/function.password-hash.php) for valid options for your default hashing algorithm.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The `hash` method will return a hashed version of the provided password.

```
$hash = Password::hash('foobar');
```

You can increase the time it takes to calculate the hash by changing the computing options.

```
$hash = Password::hash('foobar', ['cost' => 14]);
```

> Note that the length of the password hash may increase if the hashing algorithm is changed. Therefore, it is recommended to store the result in a database column that can expand beyond 60 characters (255 characters would be a good choice).
{.warning}

The `setDefaultComputingOptions` method allows you to override the default computing options.

```
Password::setDefaultComputingOptions(['cost' => 14]);
```

The `validate` method will validate hashes generated using the `hash` method.

```
$valid = Password::validate('foobar', $hash);
```

The `needsRehash` method returns `true` if the provided hash needs to be rehashed and `false` if not.

```
$needsRehash = Password::needsRehash($hash);
```

You can also set the desired computing cost when checking.

```
$needsRehash = Password::needsRehash($hash, ['cost' => 14]);
```

> Passwords will automatically be rehashed if needed upon login if you're using the [Gatekeeper](:base_url:/docs/:version:/security:authentication) authentication library.
