# Number helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The number helper contains methods that can be useful when working with numbers.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```arabic2roman``` method converts Arabic numerals to roman numerals.

	echo Num::arabic2roman(1984); // Will print "MCMLXXXIV"

> The number must be between 1 and 3999.

The ```roman2arabic``` method will convert roman numerals to Arabic numerals.

	echo Num::roman2arabic('MCMLXXXIV'); // Will print "1984"

> The number must be between I and MMMCMXCIX.
