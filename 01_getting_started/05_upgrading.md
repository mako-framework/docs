# Upgrading

* [Cache](#cache)
* [Environment variable helper](#environment_variable_helper)
* [Image processing](#image_processing)
* [Reactor](#reactor)
    - [Command registration](#reactor:command_registration)
    - [Progress bar](#reactor:progress_bar)
    - [Input](#reactor:input)
    - [Cursor](#reactor:progress_bar)
* [Routing](#routing)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 11.0.x to 12.0.x.

Mako 12.0 bumps the required PHP version to 8.5 or higher so make sure that your environment is up to date.

--------------------------------------------------------

### <a id="cache" href="#cache">Cache</a>

The deprecated `WinCache` store has been removed. You should switch to one of the other available cache stores.

--------------------------------------------------------

### <a id="environment_variable_helper" href="#environment_variable_helper">Environment variable helper</a>

The `mako\env` helper function has been rewritten to be more flexible. Previously, you could cast environment variable values to booleans using the `isBool` parameter. This parameter has now been replaced by the `as` parameter, which allows you to cast values to booleans and other types.

```
// Before

$value = mako\env('FOOBAR', isBool: true);

// Now

$value = mako\env('FOOBAR', as: Type::BOOL);
```

Read more about the new and improved `mako\env` helper [here](:base_url:/docs/:version:/getting-started:configuration#environment_aware_configuration:environment_variables).

--------------------------------------------------------

### <a id="image_processing" href="#image_processing">Image processing</a>

The old `pixl` image processing library has been completely rewritten and replaced by the new and more advanced `pixel` library. Check out the [documentation](:base_url:/docs/:version:/learn-more:image-processing) for all the details.

--------------------------------------------------------

### <a id="reactor" href="#reactor">Reactor</a>

#### <a id="reactor:command_registration" href="#reactor:command_registration">Command registration</a>

The `Command::$command` and `Command::$description` properties as well as the `Command::getArguments()` method were deprecated in Mako 11 and have been replaced by class attributes.

```
// Before

class Greet extends Command
{
	protected $command = 'greet';

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

#[CommandName('greet')]
#[CommandDescription('Prints out a greeting.')]
#[CommandArguments(
    new NamedArgument('name', alias: 'n', description: 'Name of the greetee')
]
class Greet extends Command
{
	// The rest of the command code ...
}
```

Check out the [documentation](:base_url:/docs/:version:/command-line:commands#basics:registering-commands) for all the details.

#### <a id="reactor:progress_bar" href="#reactor:progress_bar">Progress bar</a>

The `Command::progressBar()` method was also deprecated in Mako 11 and has now been removed. It should be replaced by the `Command::progress()` method. Alternatively, you can take advantage of the `Command::progressIterator()` method, which offers a more streamlined and less verbose approach to handling progress bars in your CLI applications. 

Check out the [documentation](:base_url:/docs/:version:/command-line:commands#output:components) for all the details.

#### <a id="reactor:input" href="#reactor:input">Input</a>

The deprecated `Command::question()` method has been removed, you should use the `Command::input()` method instead.

#### <a id="reactor:cursor" href="#reactor:cursor">Cursor</a>

The deprecated `Cursor::beginningOfLine()` method has been removed, you can use the `Cursor::moveToBeginningOfLine()` instead.

--------------------------------------------------------

### <a id="routing" href="#routing">Routing</a>

The syntax for registering parameters for middleware and constraints in route groups has been changed slightly:

```
// Before

$options = ['middleware' => [[CacheMiddleware::class, ['minutes' => 60]]]];

$routes->group($options, function ($routes) {

});

// Now

$options = ['middleware' => [CacheMiddleware::class => ['minutes' => 60]]];

$routes->group($options, function ($routes) {

});
```