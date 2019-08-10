# Upgrading

--------------------------------------------------------

* [Command line](#command_line)
    - [Commands](#command_line:commands)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `6.1.x` to `6.2.x`.

--------------------------------------------------------

<a id="command_line"></a>

### Command line

<a id="command_line:commands"></a>

#### Commands

> If you have documented your command and command arguments using the `$commandInformation` property and your commands aren't using positional arguments then you'll most likely be good to go until Mako `7.0`.
>
> It is still a good idea to go over your commands to make sure that they still work and update them to take full advantage of the new and vastly improved command line argument parser.

The command description has been moved to the `$description` property.

```
protected $description = 'Command description';
```

Positional arguments and options must now defined using the `getArguments` method.

```
public function getArguments(): array
{
    return
    [
        new Argument('--argument1', 'Description'),
        new Argument('--argument2', 'Description', Argument::IS_OPTIONAL),
    ];
}
```

Head over to the [documentation](:base_url:/docs/:version:/command-line:commands#input:arguments-and-options) for more information about the available flags and features.