# Date and time

--------------------------------------------------------

* [Usage](#usage) 

--------------------------------------------------------

Mako includes a DateTime class that extends the DateTime class from the PHP core. Here we'll go over the differences between the two. If you need documentation of all the default methods then you can read about them [here](http://www.php.net/manual/en/class.datetime.php).

--------------------------------------------------------

<a id="usage"></a>

### Usage

You can create a new instance using the constructor. The difference between Mako's DateTime constructor and PHP's is that the optional second parameter can be either a valid time zone string or a DateTimeZone instance.

	$time = new DateTime();

	$time = new DateTime('now', 'Europe/Paris');

	$time = new DateTime('now', new DateTimeZone('Europe/Paris'));

The ```now``` method allow you to create a DateTime instance where the time is set to "now". There's an optional parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = DateTime::now();

	$time = DateTime::now('Europe/Paris');

	$time = DateTime::now(new DateTimeZone('Europe/Paris'));

The ```createFromFormat``` method allows you to create a DateTime instance from a time string. The difference between Mako's DateTime version of the method and PHP's is that the optional second parameter can be either a valid timez one string or a DateTimeZone instance.

	$time = DateTime::createFromFormat('Y-m-d', '2014-03-28');

	$time = DateTime::createFromFormat('Y-m-d', '2014-03-28', 'Europe/Paris');

	$time = DateTime::createFromFormat('Y-m-d', '2014-03-28', new DateTimeZone('Europe/Paris'));

The ```createFromDate``` method allows you to create a DateTime instance using a date. Only the first parameter (year) is required. It'll use the current month and day if not specified. There's also an optional fourth parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = DateTime::createFromDate(2014);

	$time = DateTime::createFromDate(2014, 4);

	$time = DateTime::createFromDate(2014, 4, 28);

	$time = DateTime::createFromDate(2014, 4, 28, 'Europe/Paris');

	$time = DateTime::createFromDate(2014, 4, 28, new DateTimeZone('Europe/Paris'));

The ```createFromTimestamp``` method allows you to create a DateTime instance using a UNIX timestamp. There's an optional second parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = DateTime::createFromTimestamp($timestamp);

	$time = DateTime::createFromTimestamp($timestamp, 'Europe/Paris');

	$time = DateTime::createFromTimestamp($timestamp, new DateTimeZone('Europe/Paris'));

The ```createFromDOSTimestamp``` method allows you to create a DateTime instance using a DOS timestamp. There's an optional second parameter that accepts a valid time zone string or a DateTimeZone instance.

	$time = DateTime::createFromDOSTimestamp($timestamp);

	$time = DateTime::createFromDOSTimestamp($timestamp, 'Europe/Paris');

	$time = DateTime::createFromDOSTimestamp($timestamp, new DateTimeZone('Europe/Paris'));

The ```setTimeZone``` method allows you to change the timzone after instance creation. The timezone parameter accepts either a valid time zone string or a DateTimeZone instance.

	$time->setTimeZone('Europe/Paris');

	$time->setTimeZone(new DateTimeZone('Europe/Paris'));

The ```getTimeZones``` method retuns an array consisting of all avaiable time zones where the key is a valid time zone string while the value is a presentable name.

	$timeZones = DateTime::getTimeZones();

The ```getGroupedTimeZones``` method retuns an array consisting of grouped time zones where the key is a valid PHP time zone string while the value is a presentable name.

	$timeZones = DateTime::getGroupedTimeZones();

The ```forward``` method moves you forward in time by **x** seconds.

	$time->forward(60);

The ```rewind``` method moves you backward in time by **x** seconds.

	$time->rewind(60);

The ```getDOSTimestamp``` method returns the DOS timestamp of the DateTime instance.

	$dosTimestamp = $time->getDOSTimestamp();

The ```isLeapYear``` method returns TRUE if the year of the DateTime instance is a leap year and FALSE if not.

	$isLeapYear = $time->isLeapYear();

The ```daysInMonth``` method returnst the number of days in the month of the DateTime instance. The method also takes into account whether it is a leap year or not.

	$daysInMonth = $time->daysInMonth();