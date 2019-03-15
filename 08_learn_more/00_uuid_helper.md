# UUID helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The UUID class contains methods used to validate and generate [UUIDs](http://en.wikipedia.org/wiki/Universally_unique_identifier).

--------------------------------------------------------

<a id="usage"></a>

### Usage

The `validate` method checks if a string is a valid UUID. It will return `true` if is does and `false` if not.

```
$valid = UUID::validate('f47ac10b-58cc-4372-a567-0e02b2c3d479'); // "true"

$valid = UUID::validate('x47ac10b-58cc-4372-a567-0e02b2c3d479'); // "false"
```

The `v3` method will generate and return a version 3 UUID.

```
// The namespace must be a valid UUID

$uuid = UUID::v3('f47ac10b-58cc-4372-a567-0e02b2c3d479', 'foobar');

// You can also use one of the predefined namespaces (DNS, URL, OID and X500)

$uuid = UUID::v3(UUID::OID, 'foobar');
```

The `v4` method will return a version 4 UUID.

```
$uuid = UUID::v4();
```

The `v5` method will return a version 5 UUID.

```
// The namespace must be a valid UUID

$uuid = UUID::v5('f47ac10b-58cc-4372-a567-0e02b2c3d479', 'foobar');

// You can also use one of the predefined namespaces (DNS, URL, OID and X500)

$uuid = UUID::v5(UUID::OID, 'foobar');
```

The `sequential` methods lets you generate sequential UUIDs where the first 48 bit are reserved for the timestamp.

```
$uuid = UUID::sequential();
```

> Note that the the timestamp part of the UUID is only precise down to 10 microseconds so if you generate UUIDs in a loop where nothing else happens then you might end up with values that are not sequential (they should still be unique though).

The `toBinary` method converts a UUID from a hexadecimal representation to binary.

```
$binary = UUID::toBinary('f47ac10b-58cc-4372-a567-0e02b2c3d479');
```

The `toHexadecimal` method converts a binary representation of a UUID to a hexadecimal representation.

```
$hexadecimal = UUUID::toHexadecimal($binary);
```
