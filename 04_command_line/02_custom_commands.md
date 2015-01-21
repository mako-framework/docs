# Custom tasks

--------------------------------------------------------

* [Basics](#basics)
	- [Getting started](#basics:getting-started)
	- [Registering commands](#basics:registering-commands)
	- [Arguments and options](#basics:arguments-and-options)
* [Input](#input)
* [Output](#output)
	- [STDOUT](#output:stdout)
	- [STDERR](#output:stderr)
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

You'll also want to tell users (or remind yourself) what the command does. This is easily done using the ```$commandInformation``` property.

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

They are registered in the ```app/config/application.php``` configuration file. The array key is the name of your command and the value is the comand class name. Check out the [this page](:base_url:/packages:packages#commands) of the documentation to see how you register your custom commands in packages.

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

You can now execute your task from the command line.

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

You can now execute your task from the command line.

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

You can now execute your task from the command line.

	php reactor hello --shout

--------------------------------------------------------

<a id="input"></a>

### Input

The ```param``` method lets your read parameters from the command line.

	// php reactor mytask.test --foo=bar

	$foo = $this->input->param('foo');

	// You can also define a default value in case the parameter isn't set (default is NULL)

	$foo = $this->input->param('foo', false);

The ```text``` method will prompt the user for text input.

	$username = $this->input->text('Your username:');

	// You can also define a default value if the user doesn't write anything (default is NULL)

	$username = $this->input->text('Your username':, false);

The ```password``` method will prompt the user for hidden input.

	$password = $this->input->password('Your password');

The ```confirm``` method will prompt the user for confirmation.

	$ok = $this->input->confirm('Do you want to delete everyting?');

	// You can also set a default value (default is TRUE)

	$ok = $this->input->confirm('Do you want to delete everyting?', false);

--------------------------------------------------------

<a id="output"></a>

### Output

<a id="output:stdout"></a>

#### STDOUT

The ```write``` method writes output to STDOUT.

	$this->output->write('Hello, world!');

The ```writenl``` method writes output to STDOUT and appends a newline.

	$this->output->writenl('Hello, world!');

The ```nl``` method writes a newline to STDOUT.

	$this->output->nl();

	// You can also output multiple newlines

	$this->output->nl(5);

The ```table``` method makes it easy to render tables.

	$headers = ['Header 1', 'Header 2'];

	$tableBody = [['foo', 'bar'], ['baz', 'bax']];

	$this->output->table($headers, $tableBody);

The ```clearScreen``` method will clear the screen of all output.

	$this->output->clearScreen();

The ```beep``` will make your system beep.

	$this->output->beep();

	// You can also tell it to beep multiple times

	$this->output->beel(5);

The ```progress``` method lets you render a pretty progess bar.

	$itemCount = 1000;

	$progress = $this->output->progress($itemCount);

	for($i = 0; $i < $itemCount; $i++)
	{
		$progress->advance();
	}

	$progress->finish();

<a id="output:stderr"></a>

#### STDERR

You can write to STDERR using the ```error``` method.

	$this->output->error('Something went wrong ...');

You can also access the STDERR stream using the ```stderr``` method. It has all the same methods as STDOUT except for ```clearScreen```, ```beep``` and ```progess```.

	$this->output->stderr()->writeln('Something went wrong ...');

--------------------------------------------------------

<a id="dependency_injection"></a>

### Dependency injection

Tasks are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). This makes it easy to inject your dependencies using the constructor.

	class MyTask extends Task
	{
		protected $config;

		public function __construct(Input $input, Output $output, Config $config)
		{
			parent::__construct($input, $output);

			$this->config = $config;
		}
	}

> Note that tasks expect the first two constructor parameters to be instances of the ```Input``` and ```Output``` classes.

Tasks are also ```container aware```. You can read more about what this means [here](:base_url:/docs/:version:/getting-started:dependency-injection#container-aware).