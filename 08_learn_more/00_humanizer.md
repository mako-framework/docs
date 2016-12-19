# Humanizer

--------------------------------------------------------

* [Usage](#usage)
	- [Files](#usage:files)
	- [Dates](#usage:dates)

--------------------------------------------------------

The humanizer class helps you bring a human touch to your application.

--------------------------------------------------------

<a id="usage"></a>

### Usage

<a id="usage:files"></a>

#### Files

The ```fileSize``` method converts a file size in bytes to a more human friendly format.

	$this->humanizer->fileSize(1024); // Will return "1 KiB"

It will return binary suffixes by default can also make it output file sizes using decimal suffixes by setting the optional second parameter to FALSE.

	$this->humanizer->fileSize(1024, false); // Will return "1.02 KB"

<a id="usage:dates"></a>

#### Dates

The ```day``` method will return a human friendly representation of the day. If the day isn't within the **yesterday/today/tomorrow** range then it'll return the date in the format specified by the optional second parameter.

	$this->humanizer->day(Time::now()); // Will return "today"

	$this->humanizer->day(Time::now()->rewind(60*60*24)); // Will return "yesterday"

	$this->humanizer->day(Time::now()->forward(60*60*24)); // Will return "tomorrow"

The ```time``` method will return a human friendly representation of the time. If the time isn't within the **yesterday/today/tomorrow** range then it'll return the time in the format specified by the optional second parameter. There is also a third parameter that lets you set the clock format.

	$this->humanizer->time(Time::now()); // Will return "a minute ago"

	$this->humanizer->time(Time::now()->rewind(60*4)); // Will return "4 minutes ago"

	$this->humanizer->time(Time::now()->rewind(60*60)); // Will return "today, 08:43"

	$this->humanizer->time(Time::now()->rewind(60*60*24)); // Will return "yesterday, 08:43"

	$this->humanizer->time(Time::now()->forward(60*4)); // Will return "in 4 minutes"

	$this->humanizer->time(Time::now()->forward(60*60)); // Will return "today, 10:43"

	$this->humanizer->time(Time::now()->forward(60*60*24)); // Will return "tomorrow, 10:43"


> Both the ```day``` and ```time``` methods use the [I18n](:base_url:/docs/:version:/learn-more:internationalization) class so you can translate the strings to your own language if needed.
