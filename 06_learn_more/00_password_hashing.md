# Password hashing

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

Using md5 or sha1 hashes for storing passwords is not recommended, as they are easy to brute-force with modern hardware. The password hashing class makes it easy to hash and validate your passwords using bcrypt.

Bcrypt is a cryptographic hash function for passwords that incorporates a salt to protect against rainbow table attacks. Besides incorporating a salt to protect against rainbow table attacks, bcrypt is an adaptive hash: over time it can be made slower and slower so it remains resistant to specific brute-force search attacks against the hash and the salt.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```hash``` method will return a 60 characters long bcrypt hash.

You can increase the time it takes to calculate the hash by increasing the value of the cost parameter (only values between 4 and 31 are accepted).

	$hash = Password::hash('foobar');

	// You can increase the time it will take to compute the hash
	// by increasing the value of the $cost parameter

	$hash = Password::hash('foobar', 14);

The ```validate``` method will validate hashes generated using the ```hash``` method.

	$valid = Password::validate('foobar', $hash))

The optional third parameter is useful if you're migrating from a different type of password hash. In this example the method will now be able to validate both bcrypt and md5 hashes.

	$valid = Password::validate('foobar', $hash, function($password, $hash)
	{
		return md5($password) === $hash;
	});

The ```isLegacyHash``` method returns TRUE if the provided hash is not a bcrypt hash.

	$isLegacyHash = Password::isLegacyHash(md5('foobar')); // TRUE

	$isLegacyHash = Password::isLegacyHash(Password::hash('foobar')); // FALSE