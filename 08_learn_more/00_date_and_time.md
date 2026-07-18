# Date and time

--------------------------------------------------------

* [Date and time](#date_and_time)
    - [TimeImmutable specific methods](#time:timeimmutable_specific_methods)
    - [Time specific methods](#time:time_specific_methods)
* [Time zones](#time_zones)

--------------------------------------------------------

Mako includes a couple of classes that extends the `DateTimeImmutable`, `DateTime` and `DateTimeZone` classes from the PHP core. 

We'll only go over our customizations in this document. If you need documentation for all the methods defined in the base classes then you can read about them [here](https://php.net/manual/en/class.datetimeimmutable.php), [here](https://php.net/manual/en/class.datetime.php) and [here](https://php.net/manual/en/class.datetimezone.php).

--------------------------------------------------------

### <a id="date_and_time" href="#date_and_time">Date and time</a>

You can create a new instance using the constructor. While the first parameter behaves the same as in PHP's native date/time constructors, the optional second parameter accepts either a valid time zone string or a `DateTimeZone` instance.

> There is also a mutable class called `Time` that implements the same methods with a [couple of exceptions](#time:time_specific_methods). We recommend using `TimeImmutable`, as immutable date and time objects help prevent accidental modifications and make it easier to reason about values as they are passed around your application.

```php
$time = new TimeImmutable;

$time = new TimeImmutable('now', 'Europe/Paris');

$time = new TimeImmutable('now', new DateTimeZone('Europe/Paris'));
```

The `now` method allow you to create a `TimeImmutable` instance where the time is set to "now". There's an optional parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```php
$time = TimeImmutable::now();

$time = TimeImmutable::now('Europe/Paris');

$time = TimeImmutable::now(new DateTimeZone('Europe/Paris'));
```

The `createFromFormat` method allows you to create a `TimeImmutable` instance from a time string. Unlike PHP's `DateTimeImmutable` implementation, Mako's `TimeImmutable` class accepts either a valid timezone string or a `DateTimeZone` instance as the optional third parameter. The method returns `false` if date parsing fails.

```php
$time = TimeImmutable::createFromFormat('Y-m-d', '2014-03-28');

$time = TimeImmutable::createFromFormat('Y-m-d', '2014-03-28', 'Europe/Paris');

$time = TimeImmutable::createFromFormat('Y-m-d', '2014-03-28', new DateTimeZone('Europe/Paris'));
```

The `createFromFormatOrThrow` method works like `createFromFormat`, but throws a `ChronoException` if date parsing fails.

```php
$time = TimeImmutable::createFromFormatOrThrow('Y-m-d', '2014-03-28');

$time = TimeImmutable::createFromFormatOrThrow('Y-m-d', '2014-03-28', 'Europe/Paris');

$time = TimeImmutable::createFromFormatOrThrow('Y-m-d', '2014-03-28', new DateTimeZone('Europe/Paris'));
```

The `createFromDate` method allows you to create a `TimeImmutable` instance using a date. Only the first parameter (year) is required. It'll use the current month and day if not specified. There's also an optional fourth parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```php
$time = TimeImmutable::createFromDate(2014);

$time = TimeImmutable::createFromDate(2014, 4);

$time = TimeImmutable::createFromDate(2014, 4, 28);

$time = TimeImmutable::createFromDate(2014, 4, 28, 'Europe/Paris');

$time = TimeImmutable::createFromDate(2014, 4, 28, new DateTimeZone('Europe/Paris'));
```

The `createFromTimestamp` method allows you to create a `TimeImmutable` instance using a UNIX timestamp. There's an optional second parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```php
$time = TimeImmutable::createFromTimestamp($timestamp);

$time = TimeImmutable::createFromTimestamp($timestamp, 'Europe/Paris');

$time = TimeImmutable::createFromTimestamp($timestamp, new DateTimeZone('Europe/Paris'));
```

The `createFromDOSTimestamp` method allows you to create a `TimeImmutable` instance using a DOS timestamp. There's an optional second parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```php
$time = TimeImmutable::createFromDOSTimestamp($timestamp);

$time = TimeImmutable::createFromDOSTimestamp($timestamp, 'Europe/Paris');

$time = TimeImmutable::createFromDOSTimestamp($timestamp, new DateTimeZone('Europe/Paris'));
```

The `setTimezone` method allows you to change the timezone after instance creation. The timezone parameter accepts either a valid time zone string or a `DateTimeZone` instance.

```php
$time = $time->setTimezone('Europe/Paris');

$time = $time->setTimezone(new DateTimeZone('Europe/Paris'));
```

The `forward` method moves you forward in time by **x** seconds.

```php
$time = $time->forward(60);
```

The `rewind` method moves you backward in time by **x** seconds.

```php
$time = $time->rewind(60);
```

The `getDOSTimestamp` method returns the DOS timestamp of the `Time` instance.

```php
$dosTimestamp = $time->getDOSTimestamp();
```

The `isLeapYear` method returns `true` if the year of the `Time` instance is a leap year and `false` if not.

```php
$isLeapYear = $time->isLeapYear();
```

The `daysInMonth` method returns the number of days in the month of the `Time` instance. The method also takes into account whether it is a leap year or not.

```php
$daysInMonth = $time->daysInMonth();
```

The `daysInMonths` method returns an array containing the number of days in each month of the `Time` instance. The method also takes into account whether it is a leap year or not.

```php
$daysInMonths = $time->daysInMonths();
```

#### <a id="time:timeimmutable_specific_methods" href="#time:timeimmutable_specific_methods">TimeImmutable specific methods</a>

The `toMutable` method returns a `Time` instance set to the same time.

```php
$mutable = $time->toMutable();
```

The `toNative` method returns a `DateTime` instance set to the same time.

```php
$mutable = $time->toNative();
```

#### <a id="time:time_specific_methods" href="#time:time_specific_methods">Time specific methods</a>

The `toImmutable` method returns a `TimeImmutable` instance set to the same time.

```php
$immutable = $time->toImmutable();
```

The `toNative` method returns a `DateTimeImmutable` instance set to the same time.

```php
$mutable = $time->toNative();
```

--------------------------------------------------------

### <a id="time_zones" href="#time_zones">Time zones</a>

The `getTimezones` method returns an array consisting of all available time zones where the key is a valid time zone string while the value is a presentable name.

```php
$timeZones = TimeZone::getTimezones();
```

The `getGroupedTimezones` method returns an array consisting of grouped time zones where the key is a valid PHP time zone string while the value is a presentable name.

```php
$timeZones = TimeZone::getGroupedTimezones();
```
