# Commands

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting-started)
	- [Registering commands](#basics:registering-commands)
		- [Automatic registration](#basics:registering-commands:automatic-registration)
		- [Manual registration](#basics:registering-commands:manual-registration)
		- [Registration of package commands](#basics:registering-commands:registration-of-package-commands)
* [Input](#input)
	- [Arguments](#input:arguments)
		- [Options](#input:arguments:options)
	- [Interactive input](#input:interactive_input)
* [Output](#output)
	- [Basics](#output:basics)
	- [Components](#output:components)
	- [Formatting](#output:formatting)
* [Calling commands from commands](#calling_commands_from_commands)
* [Signal handling](#signal_handling)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The Mako command line tool comes with a set of useful commands but you can also create your own.

--------------------------------------------------------

### <a id="basics" href="#basics">Basics</a>

#### <a id="basics:getting-started" href="#basics:getting-started">Getting started</a>

All commands must extend the `mako\reactor\Command` base command and implement the `execute` method.

```
<?php

namespace app\console\commands;

use mako\reactor\attributes\CommandDescription;
use mako\reactor\attributes\CommandName;
use mako\reactor\Command;

#[CommandName('hello')]
#[CommandDescription('Prints out "Hello, World!".')]
class Hello extends Command
{
	public function execute()
	{
		$this->write('Hello, World!');
	}
}
```

> Note that the `CommandName` attribute is only required when using automatic command registration.

> You can return a custom exit code from your command's `execute` method. The default code if none is returned is `0`.

#### <a id="basics:registering-commands" href="#basics:registering-commands">Registering commands</a>

You'll have to register your command with the reactor command line tool before you can use it and you can choose between automatic or manual registration.

> You can use a combination of automatic and manual registration but we recommend choosing one method.

##### <a id="basics:registering-commands:automatic-registration" href="#basics:registering-commands:automatic-registration">Automatic registration</a>

<a id="basics:registering-commands:manual-registration"></a>

If you add the `CommandName` attribute to your command class as shown in the example above then you can let Mako register the commands for you at runtime. Mako will look for commands in the `app/console/commands` directory tree but you can override this by setting the `commands_directory` config key of the `app/config/application.php` configuration file to your desired value.

```
'commands_directory' => MAKO_APPLICATION_PATH . '/console/commands',
```

##### Manual registration

If you want to manually register your commands then you'll have to do so in the `app/config/application.php` configuration file. The array key is the name of your command and the value is the command class name.

```
'commands' => [
	'hello' => app\console\commands\Hello::class,
],
```

##### <a id="basics:registering-commands:registration-of-package-commands" href="#basics:registering-commands:registration-of-package-commands">Registration of package commands</a>

Check out [this page](:base_url:/docs/:version:/packages:packages#commands) of the documentation to see how you register your custom commands in packages.

--------------------------------------------------------

### <a id="input" href="#input">Input</a>

#### <a id="input:arguments" href="#input:arguments">Arguments</a>

You'll most likely want to pass arguments to your commands and to do so you'll have to define them using the `CommandArguments` attribute.

```
#[CommandArguments(
	new NamedArgument('option1', description: 'This is an option argument'),
	new NamedArgument('option2', alias: 'o', description: 'This is an option argument with an alias'),
	new NamedArgument('option3', alias: 'O', description: 'This is an optional option argument'),
	new PositionalArgument('argument', description: 'This is a positional argument'),
)]
```

> Note that there are 4 reserved argument names: `command`, `--env`, `--help` and `--mute`.

Next you'll have to add matching camel cased arguments to your `execute` method.

```
public function execute(string $option1, string $option2, string $option3, string $argument)
{
	// ...
}
```

You can now pass values to your command like this:

```
php reactor command "argument value" --option1 "option1 value" -O "option3 value" -o "option2 value"
```
{.language-none}

##### <a id="input:arguments:options" href="#input:arguments:options">Options</a>

You can customize how your arguments behave by using the following options:

| Option                  | Description                                                                      |
|-----------------------|----------------------------------------------------------------------------------|
| Argument::IS_OPTIONAL | The argument is considered optional                                              |
| Argument::IS_BOOL     | The argument is considered to be a boolean flag                                  |
| Argument::IS_ARRAY    | The argument is considered to be an array and subsequent values will be appended |
| Argument::IS_INT      | The argument will only accept values that can be cast to an integer              |
| Argument::IS_FLOAT    | The argument will only accept values that can be cast to a float                 |

You can also make your own custom combination of options:

```
new NamedArgument(
	'arg',
	alias: 'a',
	description: Description',
	options: Argument::IS_OPTIONAL | Argument::IS_ARRAY | Argument::IS_INT
	default: []
);
```

> Note that boolean arguments are set as optional by default and the value will automatically be set to `false` if not used.

Adding the `PromptForMissingArguments` attribute to your commands makes them prompt the user for any missing required command-line arguments. To disable prompting, pass the `--non-interactive` argument. This is useful when running the command as a cron job to prevent it from hanging due to a missing argument.

#### <a id="input:interactive_input" href="#input:interactive_input">Interactive input</a>

The `input` method lets you prompt the user for input.

```
$input = $this->input('How old are you?');
```

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is `null`.

```
$input = $this->input('How old are you?', 25);
```

The `secret` method lets you ask the user for hidden input.

```
$input = $this->secret('Password:');
```

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is `null`.

```
$input = $this->secret('Password:', false);
```

> The `secret` method will throw a `RuntimeException` if its unable to hide the user input. You can make the method fall back to non-hidden input by passing `true` to the optional third parameter.

The `confirm` method lets you ask the user to confirm their action.

```
if ($this->confirm('Do you want to delete all your files?')) {
	// Delete all files
}
```

The default answer is `No` (false) but you can choose to make `Yes` (true) the default answer.

```
if ($this->confirm('Do you want to keep all your files?', default: true)) {
	// Keep all files
}
```

The select method lets you present the user with a list of choices. By default, it returns the key of the chosen option. However, you can configure it to return the value instead by setting `returnKey` to `false`. To allow multiple selections, set `allowMultiple` to `true`.

```
$selection = $this->select('What do you want to do?', [
	'Execute order 66',
	'Open the pod bay doors',
	'Play tic-tac-toe',
]);
```

If you're passing a set of objects to `select`, you can define a custom option formatter using the `optionFormatter` argument.

```
$selection = $this->select('What do you want to do?', [
	(object) ['label' => 'Execute order 66'],
	(object) ['label' => 'Open the pod bay doors'],
	(object) ['label' => 'Play tic-tac-toe'],
], optionFormatter: static fn ($option) => $option->label);
```

--------------------------------------------------------

### <a id="output" href="#output">Output</a>

#### <a id="output:basics" href="#output:basics">Basics</a>

The `write` method lets you write output.

```
$this->write('Hello, World!');
```

The method writes to `STDOUT` by default but you can make it write to `STDERR` like this.

```
$this->write('Hello, World!', Output::ERROR);
```

There's also a handy `error` method that lets you write to `STDERR`.

```
$this->error('Hello, World!');
```

The `nl` method writes a newline to the output.

```
$this->nl();
```

The `clear` method lets you clear all output from the terminal window.

```
$this->clear();
```

#### <a id="output:components" href="#output:components">Components</a>

The `bell` method triggers the terminal bell, providing an audible alert or visual cue (depending on terminal settings) to signal a specific event. This can be particularly useful for drawing attention to important moments in a process, such as task completion, errors, or user prompts requiring immediate attention.

```
$this->bell();
```

You can also make it ring multiple times if you want to.

```
$this->bell(3);
```

The `countdown` method will print a countdown that disappears after n seconds.

```
$this->countdown(5);
```

The `progress` method allows you to display a progress bar, which is particularly useful when your command is processing a known number of items. By giving real-time feedback, it helps users track the operation's progress, providing a more engaging and user-friendly experience.

```
$items = 100;

$progress = $this->progress($items);

for ($i = 0; $i < $items; $i++) {
	$progress->advance();
}
```

The `progressIterator` method provides a convenient way to display a progress bar as you iterate over an array or any object that implements both `Traversable` and `Countable`. This method simplifies the process by automatically updating the progress bar with each iteration, ensuring that the progress accurately reflects the position within the collection.

```
$progress = $this->progressIterator(range(1, 100));

foreach($progress as $key => $value) {

}
```

The `spinner` method displays an animated spinner, which is ideal for tasks that have an unknown or variable completion time. This visual indicator provides feedback that a process is actively running, keeping users informed and reducing the perception of wait time. The spinner can enhance the user experience in situations where a task's duration is uncertain, such as network requests, data loading, or complex computations. By signaling that the task is in progress, it improves engagement and helps maintain the interface’s responsiveness.

```
$this->spinner('Processing data', function () {
	// Process your data here
});
```

> Note that the spinner will only animate when you have the `pcntl` and `posix` extensions enabled. It will gracefully degrade to a static message if the extensions are missing.

The `table` method allows you to generate and display well-formatted ASCII tables, making it easy to present tabular data directly in the console. This feature is especially useful for commands that output multiple columns of related information, such as lists of users, configuration settings, or task statuses.

```
$this->table(['Col1', 'Col2'], [['R1 C1', 'R1 C2'], ['R2 C1', 'R2 C2']]);
```

The code above will result in a table looking like this.

```
┏━━━━━━━━━━━━━━━┓
┃ Col1  ┃ Col2  ┃
┣━━━━━━━╋━━━━━━━┫
┃ R1 C1 ┃ R1 C2 ┃
┃ R2 C1 ┃ R2 C2 ┃
┗━━━━━━━┻━━━━━━━┛
```
{.language-none}

The `ol` method enables you to generate and display an ordered list, perfect for presenting sequential information or ranked data directly in the console. By numbering each item, this method provides clear organization and structure, making it easier for users to follow steps, review prioritized tasks, or understand hierarchical information.

```
$this->ol(['one', 'two', 'three', ['one', 'two'], 'four']);
```

The example above will output the following list.

```
1. one
2. two
3. three
   1. one
   2. two
4. four
```
{.language-none}

The `ul` method allows you to generate and display an unordered list, ideal for presenting collections of items where sequence or ranking isn’t necessary.

```
$this->ul(['one', 'two', 'three', ['one', 'two'], 'four']);
```

The example above will output the following list.

```
* one
* two
* three
  * one
  * two
* four
```
{.language-none}

The `alert` method lets you output alert panels that will auto-wrap your text to fit the console window. You can use one of the predefined templates (default, info, success, warning, and danger) or pass your own.

```
$this->alert('This is a success alert.', Alert::SUCCESS);

$this->alert('This is a custom alert.', '<bg_purple><white>%s</white></bg_purple>');
```

#### <a id="output:formatting" href="#output:formatting">Formatting</a>

You can format your output using formatting tags.

```
$this->write('<blue>Hello, World!</blue>');
```

You can also nest formatting tags. Just make sure to close them in the right order.

```
$this->write('<bg_green><black>Hello, World</black><yellow>!</yellow></bg_green>');
```

If you find yourself using the same nested set of formatting tags over and over again, then you'll probably want to define your own custom tags. This can be done using the `Formatter::addStyle()` method.

```
$this->output->formatter->addStyle('awesome', ['bg_green', 'black', 'blinking']);
```

Tags can also be escaped by prepending them with a backslash.

```
$this->write('\<blue>Hello, World\</blue>');
```

If you want to escape all tags in a string then you can use the `Formatter::escape()` method.

```
$escaped = $this->output->formatter->escape($string);
```

| Tag        | Description                  |
|------------|------------------------------|
| clear      | Clears all formatting styles |
| bold       | Bold text                    |
| faded      | Faded colors                 |
| underlined | Underlined text              |
| blinking   | Blinking text                |
| reversed   | Reversed text                |
| hidden     | Hidden text                  |
| black      | Black text                   |
| red        | Red text                     |
| green      | Green text                   |
| yellow     | Yellow text                  |
| blue       | Blue text                    |
| purple     | Purple text                  |
| cyan       | Cyan text                    |
| white      | white text                   |
| bg_black   | Black background             |
| bg_red     | Red background               |
| bg_green   | Green background             |
| bg_yellow  | Yellow background            |
| bg_blue    | Blue background              |
| bg_purple  | Purple background            |
| bg_cyan    | Cyan background              |
| bg_white   | white background             |

> Note that formatting will only work on linux/unix and windows consoles with ANSI support.
>
> Formatting is stripped when the output is redirected (e.g. to a log file `php reactor command > log.txt`).

--------------------------------------------------------

### <a id="calling_commands_from_commands" href="#calling_commands_from_commands">Calling commands from commands</a>

If you need to call a command from within another command then you can use the `FireTrait`.

The `fire` method executes your command in a separate process and it lets you handle the output using the optional second parameter.

```
<?php

namespace app\console\commands;

use mako\reactor\attributes\CommandName;
use mako\reactor\Command;
use mako\reactor\traits\FireTrait;

#[CommandName('proxy')]
class Proxy extends Command
{
	use FireTrait;

	public function execute()
	{
		$this->fire('hello --name=dude', function ($buffer) {
			$this->output->write($buffer);
		});
	}
}
```

If you don't want to wait for the command to finish then you can start a background process using the `fireAndForget` method.

```
<?php

namespace app\console\commands;

use mako\reactor\attributes\CommandName;
use mako\reactor\Command;
use mako\reactor\traits\FireTrait;

#[CommandName('manager')]
class Manager extends Command
{
	use FireTrait;

	public function execute()
	{
		$this->fireAndForget('worker --file=1 >> /var/log/worker');
	}
}
```

--------------------------------------------------------

### <a id="signal_handling" href="#signal_handling">Signal handling</a>

When your commands need to manage asynchronous signals, the `SignalHandler` class provides the necessary functionality. This class can be injected either through the constructor or directly within the execute method, allowing flexible integration into your command's lifecycle.

To set up specific signal handling behaviors, use the `addHandler` method to attach custom handlers for different signals. Additionally, before adding handlers, you may want to verify that your PHP installation supports signal handling by calling the `canHandleSignals` method.

```
$signalHandler->addHandler(SIGINT, function ($signal, $isLast) {
	// Handle SIGINT
});
```

The handler function receives two arguments. The first argument is the signal identifier, which indicates which signal triggered the handler. This identifier allows you to customize responses based on the specific signal received.

The second argument is a boolean flag that indicates whether this handler is the final one associated with the signal. This flag is useful if you have multiple handlers for the same signal, as it allows you to identify the last handler in the chain and perform any final cleanup or logging tasks specific to the end of signal processing.

If you want to handle multiple signals with the same logic, you can pass an array of signals when setting up the handler. This approach allows you to consolidate signal management, reducing redundancy and simplifying your code by using a single handler function for multiple signal types.

```
$signalHandler->addHandler([SIGINT, SIGTERM], function ($signal, $isLast) {
	// HANDLE SIGINT and SIGTERM
});
```

--------------------------------------------------------

### <a id="dependency_injection" href="#dependency_injection">Dependency injection</a>

Commands are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

```
<?php

namespace app\console\commands;

use mako\reactor\attributes\CommandName;
use mako\reactor\Command;

#[CommandName('hello')]
class Hello extends Command
{
	public function __construct(
		Input $input, 
		Output $output, 
		protected Config $config
	) {
		parent::__construct($input, $output);
	}
}
```

> Note that commands expect the first two constructor parameters to be instances of the `Input` and `Output` classes.

You can also inject your dependencies directly into the `execute` method since its executed by the `Container::call()` method.

```
public function execute(Config $config)
{
	$foo = $config->get('settings.foo');
}
```

Commands are also `container aware`. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).
