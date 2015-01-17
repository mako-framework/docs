# Custom tasks

--------------------------------------------------------

* [Basics](#basics)
* [Input](#input)
* [Output](#output)
	- [STDOUT](#output:stdout)
	- [STDERR](#output:stderr)
* [Dependency injection](#dependency_injection)

--------------------------------------------------------

The Mako command line tool comes with a few useful tasks but you can also create your own.

--------------------------------------------------------

<a id="basics"></a>

### Basics

All tasks must extend the ```mako\reactor\Task``` base task. The ```$taskInfo``` property is optional but it can be a helpful reminder of what your task does. Every controller has two protected properties by default, an ```input``` instance and an ```output``` instance.

	namespace app\tasks;

	use mako\reactor\Task;

	class Hello extends Task
	{
		protected static $taskInfo = array
		(
			'greet' => array
			(
				'description' => 'Displays a greeting.',
			),
		);
		
		public function greet()
		{
			$this->output->writeln('Hello, world!');
		}
	}

Passing arguments to a task action is easy as you can se in the example below.

	namespace app\tasks;

	use mako\reactor\Task;

	class Color extends Task
	{
		protected static $taskInfo = array
		(
			'favorite' => array
			(
				'description' => 'Tells you what your favorite color is.',
			),
		);

		public function favorite($color)
		{
			$this->output->writeln('Your favorite color is ' . $color);
		}
	}

You can now execute your task from the command line.

	php reactor color.favorite blue // Will print "Your favorite color is blue"

You can also make task action parameters optional by making the method parameter optional.

	public function favorite($color = 'green')
    {
        $this->output->writeln('Your favorite color is ' . $color);
    }

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