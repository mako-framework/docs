# Validation

--------------------------------------------------------

* [Usage](#usage)
	- [Basics](#usage:basics)
	- [Nested arrays](#usage:nested_arrays)
	- [Conditional rules](#usage:conditional_rules)
* [Validation rules](#validation_rules)
	- [Base rules](#validation_rules:base)
	- [Database rules](#validation_rules:database)
	- [File rules](#validation_rules:file)
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

	$rules =
	[
		'username' => ['required', 'min_length(4)', 'max_length(20)'],
		'password' => ['required'],
		'email'    => ['required', 'email'],
	];

The rules defined above will make sure that the username, password and email fields are non-empty. That the username is between 4 and 20 characters long, and that the email field contains a valid email address.

> Note that most of the included validation rules will skip validation if the field is empty. The exceptions are `required`, `one_time_token` and `token`.

Next you'll need to create a validator object. The first parameter is the input data you want to validate and the second is the set of validation rules you just defined.

	$postData = $this->request->getPost();

	$validator = $this->validator->create($postData->all(), $rules);

Now all that is left is to check if the input data is valid using either the `isValid` method or the `isInvalid` method.

	if($validator->isValid())
	{
		// Do something
	}
	else
	{
		// Display errors
	}

Retrieving error messages is done using the ```getErrors``` method.

	$errors = $validator->getErrors();

You can also use the optional ```$errors``` parameter of the ```isValid``` and ```isInvalid``` methods.

	if($validator->isValid($errors))
	{
		// Do something
	}
	else
	{
		// Display errors
	}

> An empty array will be returned if there are no errors.

<a id="usage:nested_arrays"></a>

#### Nested arrays

The validator also supports nested arrays. You can assign validation rule sets to nested fields using the "dot notation" syntax.

	$rules =
	[
		'user.email' => ['required', 'email'],
	];

You can also apply rule sets to multiple keys using wildcards.

	$rules =
	[
		'users.*.email' => ['email'],
	];

> Note that wildcard rules will only be added if the input field(s) actually exists.

<a id="usage:conditional_rules"></a>

#### Conditional rules sets

You can add rule sets to your validator instance if a certain condition is met using either the `addRules` or `addRulesIf` methods.

	$validator->addRulesIf('state', ['required', 'valid_us_state'], function() use ($postData)
	{
		return $postData->get('country') === 'United States of America';
	});

> You can also pass a boolean value instead of a closure. Rules added using either of the methods will be merged with any pre-existing rules assigned to the field.

--------------------------------------------------------

<a id="validation_rules"></a>

### Validation rules

The following validation rules are included with Mako:

<a id="validation_rules:base"></a>

#### Base rules

| Name                     | Description                                                                                                 |
|--------------------------|-------------------------------------------------------------------------------------------------------------|
| after                    | Checks that the field value is a valid date after the provided date (```after("Y-m-d","2012-09-25")```).    |
| alpha                    | Checks that the field value only contains valid alpha characters.                                           |
| alpha_dash               | Checks that the field value only contains valid alphanumeric, dash and underscore characters.               |
| alpha_dash_unicode       | Checks that the field value only contains valid alphanumeric unicode, dash and underscore characters.       |
| alpha_unicode            | Checks that the field value only contains valid alpha unicode characters.                                   |
| alphanumeric             | Checks that the field value only contains valid alphanumeric characters.                                    |
| alphanumeric_unicode     | Checks that the field value only contains valid alphanumeric unicode characters.                            |
| before                   | Checks that the field value is a valid date before the provided date (```before("Y-m-d","2012-09-25")```).  |
| between                  | Checks that the field value is between x and y (```between(5,10)```).                                       |
| date                     | Checks that the field value is a valid date (```date("Y-m-d")```).                                          |
| different                | Checks that the field value is different from the value of another field (```different("old_password")```). |
| email                    | Checks that the field value is a valid email address (uses PHP's filter_var function).                      |
| email_domain             | Checks that the field value contains a valid MX record.                                                     |
| exact_length             | Checks that the field value is of the right length (```exact_length(20)```).                                |
| float                    | Checks that the field value is a float.                                                                     |
| greater_than             | Checks that the field value is greater than x (```greater_than(5)```).                                      |
| greater_than_or_equal_to | Checks that the field value is greater than or equal to x (```greater_than_or_equal_to(5)```).              |
| hex                      | Checks that the field value is valid HEX.                                                                   |
| in                       | Checks that the field value contains one of the given values (```in(["foo","bar","baz"])```).               |
| integer                  | Checks that the field value is a integer.                                                                   |
| ip                       | Checks that the field value is an IP address (uses PHP's filter_var function).                              |
| less_than                | Checks that the field value is less than x (```less_than(5)```).                                            |
| less_than_or_equal_to    | Checks that the field value is less than or equal to x (```less_than_or_equal_to(5)```).                    |
| match                    | Checks that the field value matches the value of another field (```match("password_confirmation")```).      |
| max_length               | Checks that the field value is short enough (```max_length(20)```).                                         |
| min_length               | Checks that the field value is long enough (```min_length(10)```).                                          |
| natural                  | Checks that the field value is a natural.                                                                   |
| natural_non_zero         | Checks that the field value is a natural non zero.                                                          |
| not_in                   | Checks that the field value does not contain one of the given values (```not_in(["foo","bar","baz"])```).   |
| regex                    | Checks that the field value matches a regex pattern (```regex("/[a-z]+/i")```).                             |
| required                 | Checks that the field isn't empty.                                                                          |
| url                      | Checks that the field value is a valid URL (uses PHP's filter_var function).                                |
| uuid                     | Checks that the field value matches a valid uuid.                                                           |

<a id="validation_rules:database"></a>

#### Database rules

| Name                     | Description                                                                                |
|--------------------------|--------------------------------------------------------------------------------------------|
| exists                   | Checks that the field value exist in the database (```exists("users","email")```).         |
| unique                   | Checks that the field value doesn't exist in the database (```unique("users","email")```). |

<a id="validation_rules:file"></a>

#### File rules

| Name                     | Description                                                                                                                                                                            |
|--------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| is_uploaded              | Checks that the file is a successful upload.                                                                                                                                           |
| max_filesize             | Checks that the file is smaller or equal in size to the provided limit (`max_filesize("1MiB")` The accepted size units are `KiB`, `MiB`, `GiB`, `TiB`, `PiB`, `EiB`, `ZiB` and `YiB`). |
| mimetype                 | Checks that the file is of the specified mimetype(s) (`mimetype("image/png")` or `mimetype(["image/png", "image/jpeg"])`).                                                             |

> The `max_filesize` and `mimetype` expect `SplFileInfo` objects. The `is_uploaded` rule expects an instance of `mako\http\request\UploadedFile` (it extends `SplFileInfo`).

<a id="validation_rules:session"></a>

#### Session rules

| Name                     | Description                                                         |
|--------------------------|---------------------------------------------------------------------|
| one_time_token           | Checks that the field value matches a valid session one time token. |
| token                    | Checks that the field value matches a valid session token.          |

--------------------------------------------------------

<a id="custom_messages"></a>

### Custom messages

All error messages are defined in the ```app/i18n/*/strings/validate.php``` language file.

Adding custom field specific error messages can be done using the ```overrides.messages``` array:

	'overrides' => array
	(
		'messages' => array
		(
			'username' => array
			(
				'required' => 'You need a username!',
			),
		),
	),

You can also add custom field name translations using the ```overrides.fieldnames``` array:

	'overrides' => array
	(
		'fieldnames' => array
		(
			'email' => 'email address',
		),
	),

--------------------------------------------------------

<a id="custom_rules"></a>

### Custom rules

You can, of course, create your own custom validator rules. All rules must implement the `RuleInterface` interface.

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

If you validation rule uses parameters then it will have to implement the `WithParametersInterface` interface and if you want it to return error messages from a language file then you'll have to implement the `I18nAwareInterface` interface.

> Note that there are reusable traits that implement both interfaces so that you don't have to write the code yourself.

You can register your custom rules with the validation factory, thus making it available to all future validator instances.

	$this->validator->extend('is_foo', IsFooRule::class);

You can also register it into an existing validator instance.

	$validator->extend('is_foo', IsFooRule::class);

> Prefix the rule name with your package name and two colons (```::```) if your validator is a part of a [package](:base_url:/docs/:version:/packages:packages#configuration_i18n_and_views) to avoid naming collisions.
