# Password hashing

--------------------------------------------------------

* [Hashers](#hashers)
* [Usage](#usage)

--------------------------------------------------------

Using `md5` or `sha1` hashes for storing passwords is **not recommended** as they are easy to brute-force with modern hardware. The password hashers included with the framework make it easy to hash and verify your passwords using modern, secure and robust hashing algorithms.

--------------------------------------------------------

<a id="hashers"></a>

### Hashers

| Hasher   | Requirements                                            |
|----------|---------------------------------------------------------|
| Bcrypt   | Always available                                        |
| Argon2i  | Available if PHP has been compiled with Argon2i support |
| Argon2id | Available if PHP has been compiled with Argon2i support |

--------------------------------------------------------

<a id="usage"></a>

### Usage

We'll be using the Bcrypt hasher in all our examples but all the hashers implement the same interface.

```
$hasher = new Bcrypt;

// You can also pass an array of algorithm options

$hahser = new Bcrypt(['cost' => 14]);
```

> Check out the official [PHP documentation](https://php.net/manual/en/function.password-hash.php) for details regarding the different algorithm options.

The `create` method will return a hash of the provided password.

```
$hash = $hasher->create('foobar');
```

> Note that the length of the password hash may vary depending on the chosen hashing algorithm. Therefore, it is recommended to store the result in a database column that can expand beyond 60 characters (255 characters would be a good choice).
{.warning}

The `verify` method will validate hashes generated using the `create` method.

```
$valid = $hasher->verify('foobar', $hash);
```

The `needsRehash` method returns `true` if the provided hash needs to be rehashed and `false` if not.

```
$needsRehash = $hasher->needsRehash($hash);
```
