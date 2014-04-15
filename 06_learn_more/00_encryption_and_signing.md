# Encryption and signing

--------------------------------------------------------

* [Encryption](#encryption)
* [Signing](#signing)

--------------------------------------------------------

Mako comes with a set of classes to help you encrypt and sign your data.

> Make sure to NEVER use the example keys provided with the framework in production. ALWAYS create your own!

--------------------------------------------------------

<a id="encryption"></a>

### Encryption

The encryption library allows you to encrypt data using Mcrypt or OpenSSL.

First we'll need to get an encrypter instance. This is done using the ```CryptoManager::instance``` method.

	// Returns instance of the "default" crypto configuration defined in the config file

	$encrypter = $this->crypto->instance();

	// Returns instance of the "openssl" crypto configuration defined in the config file

	$encrypter = $this->crypto->instance('openssl');

The ```encrypt``` method is used to encrypt your data.

	$encrypted = $encrypter->encrypt('Hello, world!');

	// You can also make calls to the default encrypter through the crypto manager like this

	$encrypted = $this->crypto->encrypt('Hello, world!');

The ```decrypt``` method is used to dencrypt your data. It will return FALSE if it's unable to decrypt your data.

	$decrypted = $encrypter->decrypt('Hello, world!');

The ```encryptAndSign``` method will encrypt and sign your data. This will make sure that it has not been tampered with.

	$encryptedAndSigned = $encrypter->encryptAndSign('Hello, world!');

The ```validateAndDecrypt``` will first validate your data and then decrypt it. It will return FALSE if any of the two operations fail.

	$validatedAndDecrypted = $encrypter->validateAndDecrypt($encryptedAndSigned);

--------------------------------------------------------

<a id="signing"></a>

### Signing

First we'll need to create a signer instance.

	$signer = new Signer('secret_used_to_sign_data');

> Make sure to use a cryptographically strong secret and to keep it away from prying eyes.

The ```sign``` method returns a signed version of the provided string.

	$signed = $signer->sign('Hello, world!');

The ```validate``` method will check if your string is valid. It returns the original string if it is and FALSE if not.

	$string = $signer->validate($signed);