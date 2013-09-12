# Security helpers

--------------------------------------------------------

* [Password hashing](#password_hashing)
* [CSRF tokens](#csrf_tokens)
* [Encryption](#encryption)

--------------------------------------------------------

<a id="password_hashing"></a>

### Password hashing

Using md5 or sha1 hashes for storing passwords is not recommended, as they are easy to brute-force with modern hardware. The password hashing class makes it easy to hash and validate your passwords using bcrypt.

Bcrypt is a cryptographic hash function for passwords that incorporates a salt to protect against rainbow table attacks. Besides incorporating a salt to protect against rainbow table attacks, bcrypt is an adaptive hash: over time it can be made slower and slower so it remains resistant to specific brute-force search attacks against the hash and the salt.

The ```hash``` method will return a 60 characters long bcrypt hash.

You can increase the time it takes to calculate the hash by increasing the value of the cost parameter (only values between 4 and 31 are accepted).

	$hash = Password::hash('foobar');

	// You can increase the time it takes to compute the hash
	// by increasing the value of the $cost parameter

	$hash = Password::hash('foobar', 14);

The ```validate``` method will validate hashes generated using the ```hash``` method. It can also validate legacy hashes using a closure if the third parameter is used.

	if(Password::validate('foobar', $hash))
	{
		echo 'Valid password!';
	}
	else
	{
		echo 'Invalid password';
	}

	// If you're migrating from a different type of password hash 
	// then you can use the $legacyCheck parameter.
	// In this example the method will now be able to validate both bcrypt and md5 hashes

	$valid = Password::validate('foobar', $hash, function($password, $hash)
	{
		return md5($password) === $hash;
	});

	if($valid)
	{
		echo 'Valid password!';
	}
	else
	{
		echo 'Invalid password';
	}

The ```isLegacyHash``` method returns TRUE if the provided hash is not a bcrypt hash.

	$isLegacyHash = Password::isLegacyHash(md5('foobar')); // TRUE

	$isLegacyHash = Password::isLegacyHash(Password::hash('foobar')); // FALSE

--------------------------------------------------------

<a id="csrf_tokens"></a>

### CSRF tokens

The ```generate``` method will return a random token that can be used to prevent CSRF.

	$token = Token::generate()

The ```validate``` method will return TRUE if the token is valid and FALSE if not. It can be used as a validation rule with the input validation class.

	if(Token::validate($_POST['token']))
	{
		// valid token
	}
	else
	{
		// invalid token
	}

> Only the last 20 tokens that have been generated during a session are valid. Tokens that have been validated once are no longer considered valid.

Tokens can also be validated using the ```token``` validation rule of the [validation class](:base_url:/docs/:version:/learn-more:validation).

--------------------------------------------------------

<a id="encryption"></a>

### Encryption

The ```Crypto``` class enables to you encrypt/decrypt data using either Mcrypt or OpenSSL.

The ```factory``` method returns a cryptography object. If you want to get an instance of any of the other cryptography configurations defined in the config file then you'll have to pass the configuration name as the first argument to the factory method.

	// Returns instance of the "default" cryptography configuration defined in the config file

	$crypto = Crypto::factory();

	// Returns instance of the "openssl" cryptography configuration defined in the config file

	$crypto = Crypto::factory('openssl');

Encrypting data is done using the ```encrypt``` method.

	$encrypted = $crypto->encrypt('foobar');

	// You can also use the magic schortcut to the default configuration

	$encrypted = Crypto::encrypt('foobar');

Decrypting data is done using the ```decrypt``` method.

	$decrypted = $crypto->decrypt($encrypted);

	// You can also use the magic schortcut to the default configuration

	$decrypted = Crypto::decrypt($encrypted);