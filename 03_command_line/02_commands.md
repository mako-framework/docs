# Commands

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting-started)
	- [Registering commands](#basics:registering-commands)
	- [Arguments and options](#basics:arguments-and-options)
* [Input](#input)
	- [Helpers](#input:helpers)
* [Output](#output)
	- [Helpers](#output:helpers)
	- [Formatting](#output:formatting)
* [Calling commands from commands](#calling_commands_from_commands)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The Mako command line tool comes with a set of useful commands but you can also create your own.

--------------------------------------------------------

<a id="basics"></a>

### Basics

<a id="basics:getting-started"></a>

#### Getting started

All commands must extend the ```mako\reactor\Command``` base command and implement the ```execute``` method.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		public function execute()
		{
			$this->write('Hello, World!');
		}
	}

You might also want to tell your users (or remind yourself) what the command actually does. This can easily be done using the ```$commandInformation``` property.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		protected $commandInformation =
		[
			'description' => 'Prints a greeting.',
		];

		public function execute()
		{
			$this->write('Hello, World!');
		}
	}

You can return a custom exit code from your commands by returning an integer from the `execute` method. The default code if none is returned is `0`.

<a id="basics:registering-commands"></a>

#### Registering commands

You'll have to register your command with the reactor command line tool before you can use it.

Commands are registered in the ```app/config/application.php``` configuration file. The array key is the name of your command and the value is the command class name.

Check out the [this page](:base_url:/docs/:version:/packages:packages#commands) of the documentation to see how you register your custom commands in packages.

	'commands' =>
	[
		'hello' => 'app\console\commands\Hello',
	],

You can now call your custom command like this.

	php reactor hello

<a id="basics:arguments-and-options"></a>

#### Arguments and options

Passing arguments to a command is easy as you can se in the example below.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		public function execute($arg2)
		{
			$this->write('Hello, ' . $arg2 . '!');
		}
	}

> ```$arg0``` is the reactor script and ```$arg1``` is the name of the command.

You can now execute your command from the command line.

	php reactor hello dude

You can also use options or "named arguments".

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		public function execute($name)
		{
			$this->write('Hello, ' . $name . '!');
		}
	}

You can now execute your command from the command line.

	php reactor hello --name=dude

Options can also be used as boolean flags.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		public function execute($shout = false)
		{
			$greeting = 'Hello, World!';

			if($shout !== false)
			{
				$greeting = strtoupper($greeting);
			}

			$this->write($greeting);
		}
	}

You can now execute your command from the command line.

	php reactor hello --shout

Both arguments and options can be documented using the ```$commandInformation``` property.

	protected $commandInformation =
	[
		'description' => 'Print out a greeting.',
		'options' =>
		[
			'shout' =>
			[
				'optional'    => true,
				'description' => 'Should we output upper-case?',
			],
		],
	];

It is generally a good idea to document your arguments and options since this allows for a more user friendly error message if a required argument or option is omitted. You can also set the ```$isStrict``` property to ```TRUE``` if you want the command to fail if a non-documented argument or option is used.

--------------------------------------------------------

<a id="input"></a>

### Input

<a id="input:helpers"></a>

#### Helpers

The ```question``` method lets you ask the user for input.

	$input = $this->question('How old are you?');

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is ```NULL```.

	$input = $this->question('How old are you?', 25);

The ```secret``` method lets you ask the user for hidden input.

	$input = $this->secret('Password:');

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is ```NULL```.

	$input = $this->secret('Password:', false);

> The ```secret``` method will throw a ```RuntimeException``` if its unable to hide the user input. You can make the method fall back to non-hidden input by passing ```TRUE``` to the optional third parameter.

The ```confirm``` method lets you ask the user to confirm their action.

	if($this->confirm('Do you want to delete all your files?'))
	{
		// Delete all files
	}

The default answer is ```n``` (false) but you can choose to make ```y``` (true) the default answer.

	if($this->confirm('Do you want to delete all your files?', 'y'))
	{
		// Delete all files
	}

--------------------------------------------------------

<a id="output"></a>

### Output

<a id="output:helpers"></a>

#### Helpers

The ```write``` method lets you write output.

	$this->write('Hello, World!');

The method writes to ```STDOUT``` by default but you can make it write to ```STDERR``` like this.

	$this->write('Hello, World!', Ouput::ERROR);

There's also a handy ```error``` method that lets you write to ```STDERR```.

	$this->error('Hello, World!');

The ```nl``` method writes a newline to the output.

	$this->nl();

The ```clear``` method lets you clear all output from the terminal window.

	$this->clear();

The ```bell``` method rings the terminal bell.

	$this->bell();

You can also make it ring multiple times if you want to.

	$this->bell(3);

The ```countdown``` method will print a countdown that dissapears after n seconds.

	$this->countdown(5);

The ```progressBar``` method will let you display a nice progressbar. This is useful if your command is processing multiple items.

	$items = 100;

	$progressbar = $this->progressBar($items);

	for($i = 0; $i < $items; $i++)
	{
		$progressbar->advance();
	}

The ```table``` method lets you output a nice ASCII table.

	$this->table(['Col1', 'Col2'], [['R1 C1', 'R1 C2'], ['R2 C1', 'R2 C2']]);

This code above will result in a table looking like this.

	-----------------
	| Col1  | Col2  |
	-----------------
	| R1 C1 | R1 C2 |
	| R2 C1 | R2 C2 |
	-----------------

The ```ol``` method lets you output an ordered list.

	$this->ol(['one', 'two', 'three', ['one', 'two'], 'four']);

The example above will output the following list.

	1. one
	2. two
	3. three
	   1. one
	   2. two
	4. four

The ```ul``` method lets you output an unordered list.

	$this->ul(['one', 'two', 'three', ['one', 'two'], 'four']);

The example above will output the following list.

	* one
	* two
	* three
	  * one
	  * two
	* four

<a id="output:formatting"></a>

#### Formatting

You can format your output using formatting tags.

	$this->write('<blue>Hello, World!</blue>');

You can also nest formatting tags. Just make sure to close them in the right order.

	$this->write('<bg_green><black>Hello, World</black><yellow>!<yellow></bg_green>');

If you find yourself using the same nested set of formatting tags over and over again, then you'll probably want to define your own custom tags. This can be done using the ```Formatter::addStyle()``` method.

	$this->output->getFormatter()->addStyle('awesome', ['bg_green', 'black', 'blinking']);

Tags can also be escaped by prepending them with a backslash.

	$this->write('\<blue>Hello, World\</blue>');

If you want to escape all tags in a string then you can use the ```Formatter::escape()``` method.

	$escaped = $this->output->getFormatter()->escape($string);

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

> Formatting is stripped when the output is redirected (e.g. to a log file ```php reactor command > log.txt```).

--------------------------------------------------------

<a id="calling_commands_from_commands"></a>

### Calling commands from commands

If you need to call a command from another command then you can use the `FireTrait`.

The `fire` method executes your command in a separate process and lets you handle the output using a closure.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;
	use mako\reactor\traits\FireTrait;

	class Proxy extends Command
	{
		use FireTrait;

		public function execute()
		{
			$this->fire('hello --name=dude', function($buffer)
			{
				$this->output->write($buffer);
			});
		}
	}

If don't want to wait for the command to finish then you can start a background process using the `fireAndForget` method.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;
	use mako\reactor\traits\FireTrait;

	class Manager extends Command
	{
		use FireTrait;

		public function execute()
		{
			$this->fireAndForget('worker --file=1 >> /var/log/worker');
		}
	}

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Commands are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

	<?php

	namespace app\console\commands;

	use mako\reactor\Command;

	class Hello extends Command
	{
		protected $config;

		public function __construct(Input $input, Output $output, Config $config)
		{
			parent::__construct($input, $output);

			$this->config = $config;
		}
	}

> Note that commands expect the first two constructor parameters to be instances of the ```Input``` and ```Output``` classes.

You can also inject your dependencies directly into the ```execute``` method since its executed by the ```Container::call()``` method.

	public function execute(Config $config)
	{
		$foo = $config->get('settings.foo');
	}

Commands are also ```container aware```. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).
