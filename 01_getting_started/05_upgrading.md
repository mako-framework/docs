# Upgrading

* [Application entry points](#application_entry_points)
    - [CLI](#application_entry_points:cli)
    - [Web](##application_entry_points:web)
* [Gatekeeper](#gatekeeper)
* [Error handling](#error_handling)
* [Redis](#redis)
* [UUIDs](#uuids)
* [Deprecations](#deprecations)
    - [Reactor](#deprecations:reactor:command_registration)
        - [Command registration](#deprecations:reactor:command_registration)
        - [Progress bar](#deprecations:reactor:progress_bar)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 10.0.x to 11.0.x.

Mako 11.0 bumps the required PHP version to 8.4 or higher so make sure that your environment is up to date.

--------------------------------------------------------

### <a id="application_entry_points" href="#application_entry_points">Application entry points</a>

This is the only upgrade that is universally required for all applications. Mako 11 introduces internal changes designed to ensure compatibility with modern application servers like FrankenPHP.

#### <a id="application_entry_points:cli" href="#application_entry_points:cli">CLI</a>

Replace the contents of the `reactor` file with the following:

```
<?php

use mako\application\cli\Application;
use mako\application\CurrentApplication;

/**
 * Include the application init file.
 */
include __DIR__ . '/init.php';

/*
 * Start and run the application.
 */
CurrentApplication::set(new Application(MAKO_APPLICATION_PATH))->run();
```

#### <a id="application_entry_points:web" href="#application_entry_points:web">Web</a>

Replace the contents of the `index.php` file with the following:

```
<?php

use mako\application\CurrentApplication;
use mako\application\web\Application;

/**
 * Include the application init file.
 */
include dirname(__DIR__) . '/app/init.php';

/*
 * Start and run the application.
 */
CurrentApplication::set(new Application(MAKO_APPLICATION_PATH))->run();
```

### <a id="gatekeeper" href="#gatekeeper">Gatekeeper</a>

The ```Session::login()``` and ```Session::forceLogin()``` methods of the Gatekeeper library will now return a [`LoginStatus`](:base_url:/docs/:version:/security:gatekeeper#authentication) enum instance instead of a mix of boolean and integer values.

```
// Before (successful login)

$success = $gatekeeper->login($email, $password); // True

// Now (successful login)

$success = $gatekeeper->login($email, $password); // LoginStatus::OK
```

Check out the [documentation](:base_url:/docs/:version:/security:gatekeeper#authentication) for all the details.

### <a id="error_handling" href="#error_handling">Error handling</a>

The `ErrorHandler::handle()` and `ErrorHandler::handler()` methods have been renamed to `ErrorHandler::addHandler()` and `ErrorHandler::handle()`, respectively.

### <a id="redis" href="#redis">Redis</a>

Previously you could call multi word redis command methods using both snake case and camel case. From now on you have to use camel case.

```
// Before

$redis->config_get('*max-*-entries*');

// Now

$redis->configGet('*max-*-entries*');
```

### <a id="uuids" href="#uuids">UUIDs</a>

The `UUID::sequential()` method has been renamed to ```UUID::v4Sequential()```.

### <a id="deprecations" href="#deprecations">Deprecations</a>

The following changes are not strictly required for your application to function, but it is highly recommended to implement them. Making these updates now will help future-proof your application, reduce technical debt, and save you from potential headaches or time-consuming adjustments as your application evolves or as new versions of Mako are released.

#### <a id="deprecations:reactor" href="#deprecations:reactor">Reactor</a>

##### <a id="deprecations:reactor:command_registration" href="#deprecations:reactor:command_registration">Command registration</a>

The `Command::$command` and `Command::$description` properties as well as the `Command::getArguments()` method has been deprecated and replaced by class attributes.

```
// Before

class Hello extends Command
{
	protected $command = 'hello';

	protected $description = 'Prints out a greeting.';

    public function getArguments(): array
    {
        return [
            new Argument('-n|--name', 'Name of the greetee');
        ];
    }

    // The rest of the command code ...
}

// After

#[CommandName('hello')]
#[CommandDescription('Prints out a greeting.')]
#[CommandArguments(
    new NamedArgument('name', alias: 'n', description: 'Name of the greetee')
]
class Hello extends Command
{
	// The rest of the command code ...
}
```

Check out the [documentation](:base_url:/docs/:version:/command-line:commands#basics:registering-commands) for all the details.

##### <a id="deprecations:reactor:progress_bar" href="#deprecations:reactor:progress_bar">Progress bar</a>

The `Command::progressBar()` method has been deprecated and is no longer recommended for use. You should transition to the `Command::progress()` method. Alternatively, you can take advantage of the newly introduced `Command::progressIterator()` method, which offers a more streamlined and less verbose approach to handling progress bars in your CLI applications. 

Check out the [documentation](:base_url:/docs/:version:/command-line:commands#output:components) for all the details.