# Upgrading

--------------------------------------------------------

* [Command line](#command_line)
    - [Commands](#command_line:commands)
* [Databases](#databases)
    - [Query builder](#databases:query_builder)

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

--------------------------------------------------------

<a id="databases"></a>

### Databases

<a id="databases:query_builder"></a>

#### Query builder

Passing a `Closure` or `Query` instance to represent a subquery to the following methods is deprecated and will stop working in `7.0` (an instance of `Subquery` should be passed instead):
* `Query::table()`
* `Query::from()`
* `Query::into()`
* `Query::in()`
* `Query::notIn()`
* `Query::orIn()`
* `Query::orNotIn()`
* `Query::exists()`
* `Query::orExists()`
* `Query::notExists()`
* `Query::orNotExists()`
* `Query::union()`
* `Query::unionAll()`
* `Query::intersect()`
* `Query::intersectAll()`
* `Query::except()`
* `Query::exceptAll()`
* `Query::with()`
* `Query::withRecursive()`

All you have to do to make your application future proof is wraping your `Closure` and `Query` instances in a `Subquery` before passing them to any of the methods listed above.

```
// Before:
// SELECT * FROM (SELECT * FROM `users`) AS `mako0`

$results = $query->table(function($query)
{
    $query->table('users');
})
->all();

// After:
// SELECT * FROM (SELECT * FROM `users`) AS `users`

$results = $query->table(new Subquery(function($query)
{
    $query->table('users');
}, 'users'))
->all();
```