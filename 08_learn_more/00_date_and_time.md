# Date and time

--------------------------------------------------------

* [Time](#time)
    - [Time specific methods](#time:time_specific_methods)
    - [TimeImmutable specific methods](#time:timeimmutable_specific_methods)
* [Time zones](#time_zones)

--------------------------------------------------------

Mako includes a couple of classes that extends the `DateTime`, `DateTimeImmutable` and `DateTimeZone` classes from the PHP core. 

We'll only go over our customizations in this document. If you need documentation for all the methods defined in the base classes then you can read about them [here](https://php.net/manual/en/class.datetime.php), [here](https://php.net/manual/en/class.datetimeimmutable.php) and [here](https://php.net/manual/en/class.datetimezone.php).

--------------------------------------------------------

<a id="time"></a>

### Time

You can create a new instance using the constructor. The difference between Mako's `Time` constructor and PHP's `DateTime` constructor is that the optional second parameter can be either a valid time zone string or a `DateTimeZone` instance.

> There is also an immutable class called `TimeImmutable` that implements the same methods with a [couple of exceptions](#time:timeimmutable_specific_methods).

```
$time = new Time();

$time = new Time('now', 'Europe/Paris');

$time = new Time('now', new DateTimeZone('Europe/Paris'));
```

The `now` method allow you to create a `Time` instance where the time is set to "now". There's an optional parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```
$time = Time::now();

$time = Time::now('Europe/Paris');

$time = Time::now(new DateTimeZone('Europe/Paris'));
```

The `createFromFormat` method allows you to create a `Time` instance from a time string. The difference between Mako's `Time` class and PHP's `DateTime` class is  that the optional third parameter can be either a valid timezone string or a `DateTimeZone` instance.

```
$time = Time::createFromFormat('Y-m-d', '2014-03-28');

$time = Time::createFromFormat('Y-m-d', '2014-03-28', 'Europe/Paris');

$time = Time::createFromFormat('Y-m-d', '2014-03-28', new DateTimeZone('Europe/Paris'));
```

The `createFromDate` method allows you to create a `Time` instance using a date. Only the first parameter (year) is required. It'll use the current month and day if not specified. There's also an optional fourth parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```
$time = Time::createFromDate(2014);

$time = Time::createFromDate(2014, 4);

$time = Time::createFromDate(2014, 4, 28);

$time = Time::createFromDate(2014, 4, 28, 'Europe/Paris');

$time = Time::createFromDate(2014, 4, 28, new DateTimeZone('Europe/Paris'));
```

The `createFromTimestamp` method allows you to create a `Time` instance using a UNIX timestamp. There's an optional second parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```
$time = Time::createFromTimestamp($timestamp);

$time = Time::createFromTimestamp($timestamp, 'Europe/Paris');

$time = Time::createFromTimestamp($timestamp, new DateTimeZone('Europe/Paris'));
```

The `createFromDOSTimestamp` method allows you to create a `Time` instance using a DOS timestamp. There's an optional second parameter that accepts a valid time zone string or a `DateTimeZone` instance.

```
$time = Time::createFromDOSTimestamp($timestamp);

$time = Time::createFromDOSTimestamp($timestamp, 'Europe/Paris');

$time = Time::createFromDOSTimestamp($timestamp, new DateTimeZone('Europe/Paris'));
```

The `setTimeZone` method allows you to change the timezone after instance creation. The timezone parameter accepts either a valid time zone string or a `DateTimeZone` instance.

```
$time->setTimeZone('Europe/Paris');

$time->setTimeZone(new DateTimeZone('Europe/Paris'));
```

The `forward` method moves you forward in time by **x** seconds.

```
$time->forward(60);
```

The `rewind` method moves you backward in time by **x** seconds.

```
$time->rewind(60);
```

The `getDOSTimestamp` method returns the DOS timestamp of the `Time` instance.

```
$dosTimestamp = $time->getDOSTimestamp();
```

The `isLeapYear` method returns `true` if the year of the `Time` instance is a leap year and `false` if not.

```
$isLeapYear = $time->isLeapYear();
```

The `daysInMonth` method returns the number of days in the month of the `Time` instance. The method also takes into account whether it is a leap year or not.

```
$daysInMonth = $time->daysInMonth();
```

The `daysInMonths` method returns an array containing the number of days in each month of the `Time` instance. The method also takes into account whether it is a leap year or not.

```
$daysInMonths = $time->daysInMonths();
```

The `copy` method returns a copy of the instance.

```
$time2 = $time1->copy();

$time1->forward(30); // $time2 will be 30 seconds behind $time1
```

> The `copy` method doesn't make much sense when using the `TimeImmutable` class but it is included to keep feature parity between the two classes.

<a id="time:time_specific_methods"></a>

#### Time specific methods

The `getImmutable` method returns a `TimeImmutable` instance set to the same time.

```
$immutable = $time->getImmutable();
```

<a id="time:timeimmutable_specific_methods"></a>

#### TimeImmutable specific methods

The `getMutable` method returns a `Time` instance set to the same time.

```
$mutable = $time->getMutable();
```

--------------------------------------------------------

<a id="time_zones"></a>

### Time zones

The `getTimeZones` method returns an array consisting of all available time zones where the key is a valid time zone string while the value is a presentable name.

```
$timeZones = TimeZone::getTimeZones();
```

The `getGroupedTimeZones` method returns an array consisting of grouped time zones where the key is a valid PHP time zone string while the value is a presentable name.

```
$timeZones = TimeZone::getGroupedTimeZones();
```
