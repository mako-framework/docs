# Custom tasks

--------------------------------------------------------

* [Basics](#usage)
* [Input & output](#input_and_output)

--------------------------------------------------------

The Mako reactor tool wouldn't be useful if you couldn't create your own custom tasks. Here we'll explain you how to do so.

> All tasks must be located in the ```app/tasks``` directory and extend the mako\reactor\Task class.

--------------------------------------------------------

<a id="usage"></a>

### Basics

If you save the following code as hello.php and run it via the terminal then you'll be greeted by Hello, world!.

	<?php

	namespace app\tasks;

	class Hello extends \mako\reactor\Task
	{
		protected static $taskInfo = array
		(
			'run' => array
			(
				'description' => 'Displays a greeting.',
			),
		);
		
		public function run()
		{
			$this->cli->stdout('Hello, world!');
		}
	}

Passing arguments to a task action is easy as you can se in the example below.

	<?php

	namespace app\tasks;

	class color extends \mako\reactor\Task
	{
		protected static $taskInfo = array
		(
			'color1' => array
			(
				'description' => 'Tells you what your favorite color is.',
			),
			'color2' => array
			(
				'description' => 'Tells you what your favorite color is.',
			),
		);

		public function color1($color)
		{
			$this->cli->stdout('your favorite color is ' . $color;);
		}

		public function color2($color = 'green')
		{
			$this->cli->stdout('your favorite color is ' . $color;);
		}
	}

Calling ```php reactor color.color1 blue``` will print ```your favorite color is blue```. Trying to call the same task action without the color parameter will result in an error since its a required parameter. Calling ```php reactor color.color2``` will not result in an error since the parameter is optional but instead tell you that ```your favorite color is green```.

--------------------------------------------------------

<a id="input_and_output"></a>

### Input & output

The ```param``` method will return the value of a named task parameter.

	// php reactor files.backup --path=/foo/bar --debug

	$path = $this->cli->param('path'); // $path is now set to "/foo/bar"

	$debug = $this->cli->param('debug', false); // $debug is now set to TRUE

The ```input``` method will ask the user for input and wait until the return key has been pressed.

	$path = $this->cli->input('Path to your file'); // $path will be set to whatever the user types

The ```confirm``` method will ask the user to confirm and will only accept the values 'Y' or 'N'. The method returns TRUE if the input is 'Y' and FALSE if it's 'N'.

	$deleteFiles = $this->cli->confirm('Do you want to delete all your files?');

The ```color``` method will return the a colored string. The available colors are black, red, green, yellow, blue, purple, cyan and white.

	// "foobar" is now red

	$str = $this->cli->color('foobar', 'red');

	// "foobar" is now red on black background

	$str = $this->cli->color('foobar', 'red', 'black');

> This only works on *nix systems. On windows the string will be returned unchanged.

The ```style``` method lets you style the text string. The available styles are bold, faded, underlined, blinking, reversed and hidden.

	// "foobar is now underlined"

	$str = $this->cli->style('foobar', array('underlined'));

	// "foobar" is now underlined and blinking

	$str = $this->cli->style('foobar', array('underlined', 'blinking'));

> This only works on *nix systems. On windows the string will be returned unchanged.

The ```stdout``` method will print the message to STDOUT.

	$this->cli->stdout('Hello, world!');

	// Will print "Hello, world!" in red text on blue background

	$this->cli->stdout('Hello, world!', 'red', 'blue');

The ```stderr``` method will print the message to STDERR.

	$this->cli->stderr('Hello, world!');

	// Will print "Hello, world!" in red text on blue background

	$this->cli->stderr('Hello, world!', 'red', 'blue');

The ```newLine``` method will output n new lines.

	$this->cli->newLine(); // One new line

	$this->cli->newLine(5); // 5 new lines

The ```beep``` method will make the system "beep" n times.

	$this->cli->beep(); // Beep once

	$this->cli->beep(5); // Beep 5 times

The ```wait``` method will sleep the script for n seconds while displaying a countdown.

	$this->cli->wait() // Wait 5 seconds;

	$this->cli->wait(10, true); // Wait 10 seconds and beep every second

The ```clearScreen``` method will clear the screen.

	$this->cli->clearScreen();

The ```screenSize``` method return the size of the CLI window.

	$size = $this->cli->screenSize();

	echo $size['height'];
	echo $size['width'];

> If the method is unable to determine the size of the window then it will return an array where both values are set to 0.

The ```screenWidth``` method return the width of the CLI window.

	$width = $this->cli->screenWidth();

> If the method is unable to determine the width of the window then it will return 0.

The ```screenHeight``` method return the height of the CLI window.

	$height = $this->cli->screenHeight();

> If the method is unable to determine the height of the window then it will return 0.