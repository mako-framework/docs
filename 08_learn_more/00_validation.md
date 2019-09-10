# Validation

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage:basics)
	- [Nested arrays](#usage:nested_arrays)
	- [Conditional rules](#usage:conditional_rules)
	- [Rule builder](#usage:rule_builder)
* [Validation rules](#validation_rules)
	- [Base rules](#validation_rules:base)
	- [Database rules](#validation_rules:database)
	- [File rules](#validation_rules:file)
		- [Basic rules](#validation_rules:file:basic)
		- [Image rules](#validation_rules:file:image)
	- [Session rules](#validation_rules:session)
* [Custom messages](#custom_messages)
* [Custom rules](#custom_rules)

--------------------------------------------------------

The mako validator provides a simple and consistent way of validating user input.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:basics"></a>

#### Basics

First you'll need to define a set of rules that you want to validate your input against.

```
$rules =
[
	'username' => ['required', 'min_length(4)', 'max_length(20)'],
	'password' => ['required'],
	'email'    => ['required', 'email'],
];
```

The rules defined above will make sure that the username, password and email fields are non-empty. That the username is between 4 and 20 characters long, and that the email field contains a valid email address.

> Note that most of the included validation rules will skip validation if the field is empty. The exceptions are `required`, `one_time_token` and `token`.

Next you'll need to create a validator object. The first parameter is the input data you want to validate and the second is the set of validation rules you just defined.

```
$postData = $this->request->getPost();

$validator = $this->validator->create($postData->all(), $rules);
```

Now all that is left is to check if the input data is valid using either of the following methods: `validate`, `isValid` or `isInvalid`.

The `validate` method returns the validated input data and throws a `ValidationException` if any of the rules fail. You can retrieve the validation errors using the `ValidationException::getErrors()` method.

```
$validatedInput = $validator->validate();
```

> Note that only validated input (input fields with at least one rule) is returned. If you have a field that doesn't require any special validation then you can use the `optional` rule to ensure that it gets returned along with the validated values.
{.warning}

The `isValid` method returns `true` if the input is valid and `false` if not, while the `isInvalid` method returns `false` when the input validates and `true` when not.

```
if($validator->isValid())
{
	// Do something
}
```

Retrieving the error messages is done using the `getErrors` method

```
$errors = $validator->getErrors();
```

You can also assign it to a variable by passing it to either of the `isValid` or `isInvalid` methods.

```
if($validator->isValid($errors))
{
	// Do something
}
```

<a id="usage:nested_arrays"></a>

#### Nested arrays

The validator also supports nested arrays. You can assign validation rule sets to nested fields using the "dot notation" syntax.

```
$rules =
[
	'user.email' => ['required', 'email'],
];
```

You can also apply rule sets to multiple keys using wildcards.

```
$rules =
[
	'users.*.email' => ['email'],
];
```

> Note that wildcard rules will only be added if the input field(s) actually exists.

<a id="usage:conditional_rules"></a>

#### Conditional rules sets

You can add rule sets to your validator instance if a certain condition is met using either the `addRules` or `addRulesIf` methods.

```
$validator->addRulesIf('state', ['required', 'valid_us_state'], function() use ($postData)
{
	return $postData->get('country') === 'United States of America';
});
```

> You can also pass a boolean value instead of a closure. Rules added using either of the methods will be merged with any pre-existing rules assigned to the field.

<a id="usage:rule_builder"></a>

#### Rule builder

The validator class also comes with a handy helper method that makes it easier to build rule sets that have rules with dynamic parameters. The first parameter of the method is the name of the validation rule and any subsequent parameters are treated as rule parameters.

```
$rules =
[
	'category' => ['required', Validator::rule('in', $this->getCategoryIds())],
];

// The example above produces the same result as the following code

$rules =
[
	'category' => ['required', 'in([' . implode(',', $this->getCategoryIds()) . '])'],
];
```

--------------------------------------------------------

<a id="validation_rules"></a>

### Validation rules

The following validation rules are included with Mako:

<a id="validation_rules:base"></a>

#### Base rules

| Name                     | Description                                                                                             |
|--------------------------|---------------------------------------------------------------------------------------------------------|
| after                    | Checks that the field value is a valid date after the provided date (`after("Y-m-d","2012-09-25")`).    |
| alpha                    | Checks that the field value only contains valid alpha characters.                                       |
| alpha_dash               | Checks that the field value only contains valid alphanumeric, dash and underscore characters.           |
| alpha_dash_unicode       | Checks that the field value only contains valid alphanumeric unicode, dash and underscore characters.   |
| alpha_unicode            | Checks that the field value only contains valid alpha unicode characters.                               |
| alphanumeric             | Checks that the field value only contains valid alphanumeric characters.                                |
| alphanumeric_unicode     | Checks that the field value only contains valid alphanumeric unicode characters.                        |
| array                    | Checks that the field value is an array.                                                                |
| before                   | Checks that the field value is a valid date before the provided date (`before("Y-m-d","2012-09-25")`).  |
| between                  | Checks that the field value is between x and y (`between(5,10)`).                                       |
| date                     | Checks that the field value is a valid date (`date("Y-m-d")`).                                          |
| different                | Checks that the field value is different from the value of another field (`different("old_password")`). |
| email                    | Checks that the field value is a valid email address.                                                   |
| email_domain             | Checks that the field value contains a valid MX record.                                                 |
| exact_length             | Checks that the field value is of the right length (`exact_length(20)`).                                |
| float                    | Checks that the field value is a float.                                                                 |
| greater_than             | Checks that the field value is greater than x (`greater_than(5)`).                                      |
| greater_than_or_equal_to | Checks that the field value is greater than or equal to x (`greater_than_or_equal_to(5)`).              |
| hex                      | Checks that the field value is valid HEX.                                                               |
| in                       | Checks that the field value contains one of the given values (`in(["foo","bar","baz"])`).               |
| integer                  | Checks that the field value is a integer.                                                               |
| ip                       | Checks that the field value is an IP address (`ip`, `ip("v4")` or `ip("v6")`).                          |
| json                     | Checks that the field value contains valid JSON.                                                        |
| less_than                | Checks that the field value is less than x (`less_than(5)`).                                            |
| less_than_or_equal_to    | Checks that the field value is less than or equal to x (`less_than_or_equal_to(5)`).                    |
| match                    | Checks that the field value matches the value of another field (`match("password_confirmation")`).      |
| max_length               | Checks that the field value is short enough (`max_length(20)`).                                         |
| min_length               | Checks that the field value is long enough (`min_length(10)`).                                          |
| natural                  | Checks that the field value is a natural.                                                               |
| natural_non_zero         | Checks that the field value is a natural non zero.                                                      |
| not_in                   | Checks that the field value does not contain one of the given values (`not_in(["foo","bar","baz"])`).   |
| optional                 | This is a special validation rule that never fails.                                                     |
| regex                    | Checks that the field value matches a regex pattern (`regex("/[a-z]+/i")`).                             |
| required                 | Checks that the field isn't empty.                                                                      |
| time_zone                | Checks that the field value contains a valid PHP time zone.                                             |
| url                      | Checks that the field value is a valid URL.                                                             |
| uuid                     | Checks that the field value matches a valid uuid.                                                       |

<a id="validation_rules:database"></a>

#### Database rules

| Name                     | Description                                                                            |
|--------------------------|----------------------------------------------------------------------------------------|
| exists                   | Checks that the field value exist in the database (`exists("users","email")`).         |
| unique                   | Checks that the field value doesn't exist in the database (`unique("users","email")`). |

<a id="validation_rules:file"></a>

#### File rules

<a id="validation_rules:file:basic"></a>

##### Basic rules

| Name                     | Description                                                                                                                                                                            |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| hash                     | Checks that the file produces the expected hash (`hash("<expected_hash>")` or `hash("<expected_hash>", "<algorithm>")`).                                                               |
| hmac                     | Checks that the file produces the expected hmac (`hmac("<expected_hmac>", "key")` or `hmac("<expected_hmac>", "key", "<algorithm>")`).                                                 |
| is_uploaded              | Checks that the file is a successful upload.                                                                                                                                           |
| max_file_size            | Checks that the file is smaller or equal in size to the provided limit (`max_filesize("1MiB")` The accepted size units are `KiB`, `MiB`, `GiB`, `TiB`, `PiB`, `EiB`, `ZiB` and `YiB`). |
| mime_type                | Checks that the file is of the specified mimetype(s) (`mimetype("image/png")` or `mimetype(["image/png", "image/jpeg"])`).                                                             |

> The default hash algorithm for the `hash` and `hmac` rules is `sha256`. Any algorithm supported by [`hash_file`](https://php.net/manual/en/function.hash-file.php) can be used.

> The `max_filesize` rule expects [`SplFileInfo`](https://php.net/manual/en/class.splfileinfo.php), [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) or [`UploadedFile`](:base_url:/docs/:version:/routing-and-controllers:request#files) objects.
>
> The `hash`, `hmac`, and `mimetype` rules expect [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) or [`UploadedFile`](:base_url:/docs/:version:/routing-and-controllers:request#files) objects.
>
> The `is_uploaded` rule expects [`UploadedFile`](:base_url:/docs/:version:/routing-and-controllers:request#files) objects.

<a id="validation_rules:file:image"></a>

##### Image rules

| Name             | Description                                                                                       |
|------------------|---------------------------------------------------------------------------------------------------|
| aspect_ratio     | Checks that the image matches the expected aspect ratio (`aspect_ratio(4, 3)`).                   |
| exact_dimensions | Checks that the image matches the expected dimensions (`exact_dimensions(800, 600)`).             |
| max_dimensions   | Check that the image is smaller than or equal to the max dimensions (`max_dimensions(800, 600)`). |
| min_dimensions   | Check that the image is larger than or equal to the min dimensions (`min_dimensions(800, 600)`).  |

> The image validation rules expect [`SplFileInfo`](https://php.net/manual/en/class.splfileinfo.php), [`FileInfo`](:base_url:/docs/:version:/learn-more:file-system#file_info) or [`UploadedFile`](:base_url:/docs/:version:/routing-and-controllers:request#files) objects.

> The rules use the [`getimagesize`](https://php.net/manual/en/function.getimagesize.php) function to get the image size. You should make sure that the file you're validating is an image using the `mimetype` rule before using any of the image specific rules.
{.warning}

<a id="validation_rules:session"></a>

#### Session rules

| Name                     | Description                                                         |
|--------------------------|---------------------------------------------------------------------|
| one_time_token           | Checks that the field value matches a valid session one time token. |
| token                    | Checks that the field value matches a valid session token.          |

--------------------------------------------------------

<a id="custom_messages"></a>

### Custom messages

All error messages are defined in the `app/i18n/*/strings/validate.php` language file.

Adding custom field specific error messages can be done using the `overrides.messages` array:

```
'overrides' =>
[
	'messages' =>
	[
		'username' =>
		[
			'required' => 'You need a username!',
		],
	],
],
```

You can also add custom field name translations using the `overrides.fieldnames` array:

```
'overrides' =>
[
	'fieldnames' =>
	[
		'email' => 'email address',
	],
],
```

--------------------------------------------------------

<a id="custom_rules"></a>

### Custom rules

You can, of course, create your own custom validator rules. All rules must implement the `RuleInterface` interface.

```
<?php

use mako\validator\rules\RuleInterface;

/**
 * Is foo validation rule.
 */
class IsFooRule implements RuleInterface
{
	/**
	 * {@inheritdoc}
	 */
	public function validateWhenEmpty(): bool
	{
		return false;
	}

	/**
	 * {@inheritdoc}
	 */
	public function validate($value, array $input): bool
	{
		return mb_strtolower($value) === 'foo';
	}

	/**
	 * {@inheritdoc}
	 */
	public function getErrorMessage(string $field): string
	{
		return sprintf('The value of the %1$s field must be "foo".', $field);
	}
}
```

If you want it to return error messages from a language file then you'll have to implement the `I18nAwareInterface` interface.

> Note that there is a reusable trait (`I18nAwareTrait`) that implements the interface so that you don't have to write the code yourself.

You can register your custom rules with the validation factory, thus making it available to all future validator instances.

```
$this->validator->extend('is_foo', IsFooRule::class);
```

You can also register it into an existing validator instance.

```
$validator->extend('is_foo', IsFooRule::class);
```

> Prefix the rule name with your package name and two colons (`::`) if your validator is a part of a [package](:base_url:/docs/:version:/packages:packages#configuration_i18n_and_views) to avoid naming collisions.
