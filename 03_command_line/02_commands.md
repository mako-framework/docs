# Commands

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting-started)
	- [Registering commands](#basics:registering-commands)
		- [Automatic registration](#basics:registering-commands:automatic-registration)
		- [Manual registration](#basics:registering-commands:manual-registration)
		- [Registration of package commands](#basics:registering-commands:registration-of-package-commands)
* [Input](#input)
	- [Arguments and options](#input:arguments-and-options)
		- [Flags](#input:arguments-and-options:flags)
	- [Interactive input](#input:interactive_input)
* [Output](#output)
	- [Basics](#output:basics)
	- [Helpers](#output:helpers)
	- [Formatting](#output:formatting)
* [Calling commands from commands](#calling_commands_from_commands)
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

> You can use a combination of automatic and manual registration be we recommend choosing one method.

##### <a id="basics:registering-commands:automatic-registration" href="#basics:registering-commands:automatic-registration">Automatic registration</a>

<a id="basics:registering-commands:manual-registration"></a>

If you add the `CommandName` attribute to your command class as shown in the example above then you can let Mako register the commands for you at runtime. Mako will look for commands in the `app/console/commands` directory tree but you can override this by setting the `commands_directory` config key of the `app/config/application.php` configuration file to your desired value.

```
'commands_directory' => MAKO_APPLICATION_PATH . '/console/commands',
```

##### Manual registration

If you want to manually register your commands then you'll have to do so in the `app/config/application.php` configuration file. The array key is the name of your command and the value is the command class name.

```
'commands' =>
[
	'hello' => app\console\commands\Hello::class,
],
```

##### <a id="basics:registering-commands:registration-of-package-commands" href="#basics:registering-commands:registration-of-package-commands">Registration of package commands</a>

Check out [this page](:base_url:/docs/:version:/packages:packages#commands) of the documentation to see how you register your custom commands in packages.

--------------------------------------------------------

### <a id="input" href="#input">Input</a>

#### <a id="input:arguments-and-options" href="#input:arguments-and-options">Arguments and options</a>

You'll most likely want to pass arguments to your commands and to do so you'll have to define them using the `Arguments` attribute.

```
#[Arguments(
	new Argument('argument', 'This is a positional argument'),
	new Argument('--option1', 'This is an option argument'),
	new Argument('-o|--option2', 'This is an option argument with an alias'),
	new Argument('--option3', 'This is an optional option argument', Argument::IS_OPTIONAL),
)]
```

> Note that there are 4 reserved argument names: `command`, `--env`, `--help` and `--mute`.

Next you'll have to add matching camel cased arguments to your `execute` method.

```
public function execute(string $argument, string $option1, string $option2, string $option3 = null)
{
	// ...
}
```

You can now pass values to your command like this:

```
php reactor command "argument value" --option1="option1 value" -o "option3 value"
```

##### <a id="input:arguments-and-options:flags" href="#input:arguments-and-options:flags">Flags</a>

In the example above we made one of our arguments optional by using the `Argument::IS_OPTIONAL` flag. Below you'll see the complete list of available flags that you can use:

| Flag                  | Description                                                                      |
|-----------------------|----------------------------------------------------------------------------------|
| Argument::IS_OPTIONAL | The argument is considered optional                                              |
| Argument::IS_BOOL     | The argument is considered to be a boolean flag                                  |
| Argument::IS_ARRAY    | The argument is considered to be an array and subsequent values will be appended |
| Argument::IS_INT      | The argument will only accept values that can be cast to an integer              |
| Argument::IS_FLOAT    | The argument will only accept values that can be cast to a float                 |

You can also make your own combination of flags:

```
new Argument('--arg', 'Description', Argument::IS_OPTIONAL | Argument::IS_ARRAY | Argument::IS_INT);
```

> Note that boolean arguments are set as optional by default and the value will automatically be set to `false` if not used.

#### <a id="input:interactive_input" href="#input:interactive_input">Interactive input</a>

The `question` method lets you ask the user for input.

```
$input = $this->question('How old are you?');
```

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is `null`.

```
$input = $this->question('How old are you?', 25);
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

The default answer is `n` (false) but you can choose to make `y` (true) the default answer.

```
if ($this->confirm('Do you want to delete all your files?', 'y')) {
	// Delete all files
}
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

#### <a id="output:helpers" href="#output:helpers">Helpers</a>

The `bell` method rings the terminal bell.

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

The `progressBar` method will let you display a nice progressbar. This is useful if your command is processing multiple items.

```
$items = 100;

$progressbar = $this->progressBar($items);

for ($i = 0; $i < $items; $i++) {
	$progressbar->advance();
}
```

The `table` method lets you output a nice ASCII table.

```
$this->table(['Col1', 'Col2'], [['R1 C1', 'R1 C2'], ['R2 C1', 'R2 C2']]);
```

This code above will result in a table looking like this.

```
┏━━━━━━━━━━━━━━━┓
┃ Col1  ┃ Col2  ┃
┣━━━━━━━╋━━━━━━━┫
┃ R1 C1 ┃ R1 C2 ┃
┃ R2 C1 ┃ R2 C2 ┃
┗━━━━━━━┻━━━━━━━┛
```
{.language-none}

The `ol` method lets you output an ordered list.

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

The `ul` method lets you output an unordered list.

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
$this->write('<bg_green><black>Hello, World</black><yellow>!<yellow></bg_green>');
```

If you find yourself using the same nested set of formatting tags over and over again, then you'll probably want to define your own custom tags. This can be done using the `Formatter::addStyle()` method.

```
$this->output->getFormatter()->addStyle('awesome', ['bg_green', 'black', 'blinking']);
```

Tags can also be escaped by prepending them with a backslash.

```
$this->write('\<blue>Hello, World\</blue>');
```

If you want to escape all tags in a string then you can use the `Formatter::escape()` method.

```
$escaped = $this->output->getFormatter()->escape($string);
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
