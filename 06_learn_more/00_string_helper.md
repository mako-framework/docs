# String helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The string helper contains a collection of string manipulation methods.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```random``` method returns a random string of the selected type and length. The available constants are 'ALNUM', 'ALPHA', 'HEXDEC', 'NUMERIC' and 'SYMBOLS'. You can also combine pools or pass your own pool of characters.

> Generated strings containing characters from the 'SYMBOLS' pool should be escaped if used in HTLM and/or XML documents.

	$str = String::random(String::ALNUM);

	$str = String::random(String::ALPHA);

	$str = String::random(String::NUMERIC);

	$str = String::random(String::ALNUM . String::SYMBOLS);

	$str = String::random(String::ALNUM . 'æøåÆØÅ', 64);

The ```nl2br``` method converts newlines to ```<br>```.

	$str = String::nl2br($_POST['input']);

> The nl2br method will return HTML5 tags but you can make it return XHTML compatible tags by defining the ```MAKO_XHTML``` constant.

The br2nl method converts ```<br />``` and ```<br>``` to newlines.

	$str = String::br2nl($input);

The ```limitChars``` method will limit the number of characters in a string.

	$str = String::limitChars($input, 300);

The ```limitWords``` method will limit the number of words in a string.

	$str = String::limitWords($input, 300);

The ```increment``` method append an incremental numeric sufix to a string.

	$str = String::increment('banana'); // Will return "banana_1"
	$str = String::increment('banana_1'); // Will return "banana_2"

The ```alternator``` method will return a closure that alternates between the chosen strings.

	$alt = String::alternator(['foo', 'bar']);

	echo $alt(); // foo
	echo $alt(); // bar
	echo $alt(); // foo

The ```ascii``` method returns string where all non-ASCII characters have been stripped.

	// $str will be set to "Hello, world! It Works!"

	$str = String::ascii('Hello, world! ÆØÅ It Works!');

The ```slug``` method returns a URL friendly string.

	$str = String::slug('Hello World!'); // $str will be set to "hello-world"

The ```autolink``` method returns a string where all URLs are converted into clickable links.

	$str = String::autolink($text);

The ```mask``` method will return a masked string where n characters are visible while the rest will be masked.

	$str = String::mask('password'); // $str will be set to "*****ord"

The ```camel2underscored``` method will convert camel case to underscored.

	$str = String::camel2underscored('HelloWorld'); // $str will be set to "hello_world"

The ```underscored2camel``` method will convert underscored to camel case.

	$str = String::underscored2camel('hello_world'); // $str will be set to "helloWorld"
	$str = String::underscored2camel('hello_world', true); // $str will be set to "HelloWorld"