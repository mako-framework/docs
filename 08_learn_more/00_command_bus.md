# Command bus

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The command bus allows you to create ```commands``` that can be dispatched to corresponding ```handlers```.

The main advantage of using this pattern is that the commands can be used from anywhere within your application and thus greatly reduce code duplication.

--------------------------------------------------------

<a id="usage"></a>

### Usage

In this example we'll be creating a command and a handler for creating users.

First we'll make our ```CreateUserCommand```. As you can see, the command acts as a simple data container.

	<?php

	namespace app\commands;

	use mako\commander\CommandInterface;

	class CreateUserCommand implements CommandInterface
	{
		public $email;
		public $username;
		public $password;

		public function __construct($email, $username, $password)
		{
			$this->email    = $email;
			$this->username = $username;
			$this->password = $password;
		}
	}

Next we'll make a ```CreateUserCommandHandler```. Command handlers are instantiated by the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection) so you can easily inject all your dependencies using the constructor.


	<?php

	namespace app\commands;

	use mako\auth\Gatekeeper;
	use mako\commander\CommandHandlerInterface;

	class CreateUserCommandHandler implements CommandHandlerInterface
	{
		protected $gatekeeper;

		public function __construct(Gatekeeper $gatekeeper)
		{
			$this->gatekeeper = $gatekeeper;
		}

		public function handle(CreateUserCommand $command)
		{
			$email    = $command->email;
			$username = $command->username;
			$password = $command->password;

			$this->gatekeeper->createUser($email, $username, $password);
		}
	}

This is a very basic example but you would also want to include input validation in your command handler. The ```CommandHandlerInterface::handle()``` method can return data that can be used to handle successes and errors.

> Note that all command handlers must have the same name as the command that they're handling with a ```Handler``` suffix.

We are now ready to dispatch our command using the ```CommandBus::dispatch()``` method. In the following example we'll be creating a user from a controller method.

	<?php

	namespace app\controllers;

	use app\commands\CreateUserCommand;
	use mako\commander\CommandBus;
	use mako\http\Request;
	use mako\http\routing\Controller;

	class Register extends Controller
	{
		public function createUser(Request $request, CommandBus $commander)
		{
			$email    = $request->post('email');
			$username = $request->post('username');
			$password = $request->post('password');

			$commander->dispatch(new CreateUserCommand($email, $username, $password));
		}
	}

As previously mentioned, you can re-use your commands anywhere in your application. Here we'll dispatch our command from a console command, and as you can see we haven't had to duplicate any of the user creation code.

	<?php

	namespace app\commands\users;

	use app\commands\CreateUserCommand;
	use mako\commander\CommandBus;
	use mako\reactor\Command;

	class Create extends Command
	{
	    public function execute(CommandBus $commander)
	    {
	    	$email    = $this->question('Email:');
	    	$username = $this->question('Username:');
	    	$password = $this->secret('Password:');

	        $commander->dispatch(new CreateUserCommand($email, $username, $password));
	    }
	}