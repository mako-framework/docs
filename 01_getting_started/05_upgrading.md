# Upgrading

--------------------------------------------------------

* [Gatekeeper](#gatekeeper)
* [String helper](#string_helper)
* [Validation](#validation)
* [Other](#other)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `8.1.x` to `9.0.x`.

Mako `9.0` drops support for PHP version `7.4` and requires PHP version `8.0` or higher so make sure that your environment is up to date.

--------------------------------------------------------

<a id="gatekeeper"></a>

### Gatekeeper

The gatekeeper authorization trait has been moved from `mako\http\routing\traits\AuthorizationTrait` to `mako\gatekeeper\authorization\http\routing\traits\AuthorizationTrait` so make sure to update your use statements.

--------------------------------------------------------

<a id="string_helper"></a>

### String helper

The `Str::camel2underscored()` and `Str::underscored2camel()` helper methods have been removed and replaced by the `Str::camelToSnake()` and `Str::snakeToCamel()` methods.

```
// Before

$snakeCased = Str::camel2underscored('helloWorld');

$camelCased = Str::underscored2camel('hello_world');

// Now

$snakeCased = Str::camelToSnake('helloWorld');

$camelCased = Str::snakeToCamel('hello_world');
```

--------------------------------------------------------

<a id="validation"></a>

### Validation

#### Renamed classes

Several classes have been renamed for consistency so make sure to update your use staments.

| Before                                        | Now                                                           |
|-----------------------------------------------|---------------------------------------------------------------|
| mako\http\routing\traits\InputValidationTrait | mako\validator\input\http\routing\traits\InputValidationTrait |
| mako\validator\input\HttpInput                | mako\validator\input\http\Input                               |
| mako\validator\input\HttpInputInterface       | mako\validator\input\http\InputInterface                      |

#### Rule builder replaced by helper function

The `mako\validator\Validator::rule()` helper method has been replaced by the `mako\f()` helper function.

```
// Before

$rules =
[
	'category' => ['required', Validator::rule('in', $this->getCategoryIds())],
];

// Now

$rules =
[
	'category' => ['required', f('in', $this->getCategoryIds())],
];
```

--------------------------------------------------------

<a id="other"></a>

### Other

The following class constants have been made protected:

* mako\application\cli\commands\server\Server::MAX_PORTS_TO_TRY
* mako\classes\ClassFinder::PHP_FILENAME_PATTERN
* mako\cli\input\arguments\Argument::NAME_REGEX
* mako\cli\input\arguments\Argument::ALIAS_REGEX
* mako\cli\input\arguments\ArgvParser::INT_REGEX
* mako\cli\input\arguments\ArgvParser::FLOAT_REGEX
* mako\cli\output\formatter\Formatter::TAG_REGEX
* mako\cli\output\formatter\Formatter::ESCAPED_TAG_REGEX
* mako\cli\output\formatter\Formatter::ANSI_SGR_SEQUENCE_REGEX
* mako\cli\output\helpers\Alert::PADDING
* mako\cli\output\helpers\Countdown::SLEEP_TIME
* mako\commander\CommandBus::COMMAND_SUFFIX
* mako\commander\CommandBus::HANDLER_SUFFIX
* mako\database\midgard\ORM::PRIMARY_KEY_TYPE_INCREMENTING
* mako\database\midgard\ORM::PRIMARY_KEY_TYPE_UUID
* mako\database\midgard\ORM::PRIMARY_KEY_TYPE_CUSTOM
* mako\database\midgard\ORM::PRIMARY_KEY_TYPE_NONE
* mako\database\midgard\relations\Relation::EAGER_LOAD_CHUNK_SIZE
* mako\database\query\compilers\Compiler::JSON_PATH_SEPARATOR
* mako\error\handlers\web\DevelopmentHandler::SOURCE_PADDING
* mako\i18n\I18n::PLURALIZATION_TAG_REGEX
* mako\i18n\I18n::NUMBER_TAG_REGEX
* mako\redis\Redis::CRLF
* mako\redis\Redis::CRLF_LENGTH
* mako\redis\Redis::VERBATIM_PREFIX_LENGTH
* mako\redis\Redis::END
* mako\security\crypto\encrypters\Encrypter::DERIVATION_HASH
* mako\security\crypto\encrypters\Encrypter::DERIVATION_ITERATIONS
* mako\security\Signer::MAC_LENGTH
* mako\session\Session::MAX_TOKENS
* mako\view\compilers\Template::VERBATIM_PLACEHOLDER