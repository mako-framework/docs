# Date and time

--------------------------------------------------------

* [Usage](#usage) 

--------------------------------------------------------

Mako includes a Time class that extends the DateTime class from the PHP core. Here we'll go over the differences between the two. If you need documentation of all the default methods then you can read about them [here](http://www.php.net/manual/en/class.datetime.php).

--------------------------------------------------------

<a id="usage"></a>

### Usage

You can create a new instance using the constructor. The difference between Mako's Time constructor and PHP's DateTime constructor is that the optional second parameter can be either a valid time zone string or a DateTimeZone instance.

	$time = new Time();

	$time = new Time('now', 'Europe/Paris');

	$time = new Time('now', new DateTimeZone('Europe/Paris'));

The ```now``` method allow you to create a Time instance where the time is set to "now". There's an optional parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = Time::now();

	$time = Time::now('Europe/Paris');

	$time = Time::now(new DateTimeZone('Europe/Paris'));

The ```createFromFormat``` method allows you to create a Time instance from a time string. The difference between Mako's Time class and PHP's DateTime class is  that the optional second parameter can be either a valid timez one string or a DateTimeZone instance.

	$time = Time::createFromFormat('Y-m-d', '2014-03-28');

	$time = Time::createFromFormat('Y-m-d', '2014-03-28', 'Europe/Paris');

	$time = Time::createFromFormat('Y-m-d', '2014-03-28', new DateTimeZone('Europe/Paris'));

The ```createFromDate``` method allows you to create a Time instance using a date. Only the first parameter (year) is required. It'll use the current month and day if not specified. There's also an optional fourth parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = Time::createFromDate(2014);

	$time = Time::createFromDate(2014, 4);

	$time = Time::createFromDate(2014, 4, 28);

	$time = Time::createFromDate(2014, 4, 28, 'Europe/Paris');

	$time = Time::createFromDate(2014, 4, 28, new DateTimeZone('Europe/Paris'));

The ```createFromTimestamp``` method allows you to create a Time instance using a UNIX timestamp. There's an optional second parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = Time::createFromTimestamp($timestamp);

	$time = Time::createFromTimestamp($timestamp, 'Europe/Paris');

	$time = Time::createFromTimestamp($timestamp, new DateTimeZone('Europe/Paris'));

The ```createFromDOSTimestamp``` method allows you to create a Time instance using a DOS timestamp. There's an optional second parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = Time::createFromDOSTimestamp($timestamp);

	$time = Time::createFromDOSTimestamp($timestamp, 'Europe/Paris');

	$time = Time::createFromDOSTimestamp($timestamp, new DateTimeZone('Europe/Paris'));

The ```setTimeZone``` method allows you to change the timzone after instance creation. The timezone parameter accepts either a valid time zone string or a DateTimeZone instance.

	$time->setTimeZone('Europe/Paris');

	$time->setTimeZone(new DateTimeZone('Europe/Paris'));

The ```getTimeZones``` method retuns an array consisting of all avaiable time zones where the key is a valid time zone string while the value is a presentable name.

	$timeZones = Time::getTimeZones();

The ```getGroupedTimeZones``` method retuns an array consisting of grouped time zones where the key is a valid PHP time zone string while the value is a presentable name.

	$timeZones = Time::getGroupedTimeZones();

The ```forward``` method moves you forward in time by **x** seconds.

	$time->forward(60);

The ```rewind``` method moves you backward in time by **x** seconds.

	$time->rewind(60);

The ```getDOSTimestamp``` method returns the DOS timestamp of the Time instance.

	$dosTimestamp = $time->getDOSTimestamp();

The ```isLeapYear``` method returns TRUE if the year of the Time instance is a leap year and FALSE if not.

	$isLeapYear = $time->isLeapYear();

The ```daysInMonth``` method returnst the number of days in the month of the Time instance. The method also takes into account whether it is a leap year or not.

	$daysInMonth = $time->daysInMonth();