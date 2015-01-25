# Password hashing

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Using md5 or sha1 hashes for storing passwords is not recommended, as they are easy to brute-force with modern hardware. The password hashing class makes it easy to hash and validate your passwords using a secure method.

The password hashing class uses PHP's ```password_*``` methods internally. Passwords are currently hashed using the bcrypt algorithm (default as of PHP 5.5.0) but this may change over time as new and stronger algorithms are added to PHP.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```hash``` method will return a 60 character long hash.

	$hash = Password::hash('foobar');

You can increase the time it takes to calculate the hash by increasing the value of the ```$cost``` parameter (only values between ```4``` and ```31``` are accepted). 

	$hash = Password::hash('foobar', 14);

> Note that the length of the password hash may increase if the hashing algorithm is changed. Therefore, it is recommended to store the result in a database column that can expand beyond 60 characters (255 characters would be a good choice).

The ```setDefaultComputingCost``` method allows you to override the default computing cost (currently set to ```10```).

	Password::setDefaultComputingCost(14);

The ```validate``` method will validate hashes generated using the ```hash``` method.

	$valid = Password::validate('foobar', $hash);

The ```needsRehash``` method returns TRUE if the provided hash needs to be rehashed and FALSE if not.

	$needsRehash = Password::needsRehash($hash);

You can also set the desired computing cost when checking.

	$needsRehash = Password::needsRehash($hash, 14);

> Passwords will automatically be rehashed if needed upon login if you're using the [Gatekeeper](:base_url:/docs/:version:/security:authentication) authentication library.