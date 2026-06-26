# Upgrading
--------------------------------------------------------

* [Enum cases](#enum_cases)
* [Query builder sorting](#query_builder_sorting)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 12.1.x to 12.2.x.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### <a id="enum_cases" href="#enum_cases">Enum cases</a>

All enum cases have been converted from `UPPER_SNAKE_CASE` to `PascalCase` to follow the standard PHP convention for naming enum cases. The reason they were previously in `UPPER_SNAKE_CASE` is that many of them were converted from classes with class constants.

```php{1}
{- Status::NOT_FOUND -}
{+ Status::NotFound +}
```

All code will continue to work as is until Mako `13.0`, but we recommend making the changes now so you’ll have less work to do in the future. Here's a list of the affected enums:

- mako\cli\input\Key
- mako\database\query\SortDirection
- mako\database\query\VectorDistance
- mako\env\Type
- mako\file\Permission
- mako\gatekeeper\LoginStatus
- mako\http\response\Status
- mako\http\response\senders\stream\event\Type
- mako\pixel\image\operations\AspectRatio
- mako\pixel\image\operations\Flip
- mako\pixel\image\operations\WatermarkPosition
- mako\pixel\metadata\xmp\properties\Type

--------------------------------------------------------

### <a id="query_builder_sorting" href="#query_builder_sorting">Query builder sorting</a>

Passing the strings `ASC` and `DESC` to the following methods has been deprecated in favor of the `SortDirection` enum:

- Query::orderBy()
- Query::orderByRaw()
- Query::orderByVectorDistance()

```php{1}
{- $query->orderBy('id', 'DESC') -}
{+ $query->orderBy('id', SortDirection::Descending) +}
```

You may still use the following methods:

- Query::ascending()
- Query::descending()
- Query::ascendingRaw()
- Query::descendingRaw()
- Query::ascendingVectorDistance()
- Query::descendingVectorDistance()