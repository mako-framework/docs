# Custom commands

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
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The Mako command line tool comes with a set of useful commands but you can also create your own.

--------------------------------------------------------

<a id="basics"></a>

### Basics

<a id="basics:getting-started"></a>

#### Getting started

All commands must extend the ```mako\reactor\Command``` base command and implement the ```execute``` method.

	namespace app\commands;

	use mako\reactor\Coomand;

	class Hello extends Command
	{
		public function execute()
		{
			$this->write('Hello, World!');
		}
	}

You'll also want to tell your users (or remind yourself) what the command actually does. This is easily done using the ```$commandInformation``` property.

	namespace app\commands;

	use mako\reactor\Coomand;

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

<a id="basics:registering-commands"></a>

#### Registering commands

You'll have to register your command with the reactor command line tool before you can use it. 

Commands are registered in the ```app/config/application.php``` configuration file. The array key is the name of your command and the value is the comand class name. 

Check out the [this page](:base_url:/docs/:version:/packages:packages#commands) of the documentation to see how you register your custom commands in packages.

	'commands' => 
	[
		'hello' => 'app\commands\Hello',
	],

You can now call your custom command like this.

	php reactor hello

<a id="basics:arguments-and-options"></a>

#### Arguments and options

Passing arguments to a command is easy as you can se in the example below.

	namespace app\commands;

	use mako\reactor\Coomand;

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

	namespace app\commands;

	use mako\reactor\Coomand;

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

	namespace app\commands;

	use mako\reactor\Coomand;

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

--------------------------------------------------------

<a id="input"></a>

### Input

<a id="input:helpers"></a>

#### Helpers

The ```question``` method lets you ask the user for input.

	$input = $this->question('How old are you?');

You can also specify a default return value in the event that the user chooses not to enter anything. The default return value for empty input is ```NULL```.

	$input = $this->question('How old are you?', 25);

The ```Confirmation``` method lets you ask the user for confirmation.

	if($this->confirmation('Do you want to delete all your files?'))
	{
		// Delete all files
	}

The default answer is ```n``` (true) but you can choose to make ```y``` (false) the default answer.

	if($this->confirmation('Do you want to delete all your files?', 'y'))
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

There's also a handy ```error``` method that lets you write to ```STDERR``` by default.

	$this->error('Hello, World!');

The ```nl``` method writes a newline to the output.

	$this->nl();

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

<a id="output:formatting"></a>

#### Formatting

You can also format your output using formatting tags.

	$this->write('<blue>Hello, World!</blue>');

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

You can also nest tags like this.

	$this->write('<bg_green><black>Hello, World</black><yellow>!<yellow></bg_green>');

Tags can also be escaped by prepending them with a backlash.

	$this->write('\<blue>Hello, World\</blue>');

You can also escape all tags in a string using the ```Formatter::escape()``` method.

	$escaped = $this->output->getFormatter()->escape($string);

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Commands are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

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