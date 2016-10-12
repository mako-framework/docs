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

	$str = Str::random(Str::ALNUM);

	$str = Str::random(Str::ALPHA);

	$str = Str::random(Str::NUMERIC);

	$str = Str::random(Str::ALNUM . Str::SYMBOLS);

	$str = Str::random(Str::ALNUM . 'æøåÆØÅ', 64);

The ```nl2br``` method converts newlines to ```<br>```.

	$str = Str::nl2br($_POST['input']);

> The nl2br method will return HTML5 tags by default but you can make it return XHTML compatible tags by setting the optional second parameter to TRUE.

The br2nl method converts ```<br />``` and ```<br>``` to newlines.

	$str = Str::br2nl($input);

The ```limitChars``` method will limit the number of characters in a string.

	$str = Str::limitChars($input, 300);

The ```limitWords``` method will limit the number of words in a string.

	$str = Str::limitWords($input, 300);

The ```increment``` method append an incremental numeric sufix to a string.

	$str = Str::increment('banana'); // Will return "banana_1"
	$str = Str::increment('banana_1'); // Will return "banana_2"

The ```alternator``` method will return a closure that alternates between the chosen strings.

	$alt = Str::alternator(['foo', 'bar']);

	echo $alt(); // foo
	echo $alt(); // bar
	echo $alt(); // foo

The ```ascii``` method returns a string where all non-ASCII characters have been stripped.

	// $str will be set to "Hello, world! It Works!"

	$str = Str::ascii('Hello, world! ÆØÅ It Works!');

The ```slug``` method returns a URL friendly string.

	$str = Str::slug('Hello World!'); // $str will be set to "hello-world"

The ```autolink``` method returns a string where all URLs have been converted into clickable links.

	$str = Str::autolink($text);

The ```mask``` method will return a masked string where n characters are visible while the rest will be masked.

	$str = Str::mask('password'); // $str will be set to "*****ord"

The ```pluralzie``` method will return the pluralized form of a singular word.

	$str = Str::pluralize('apple');

> The pluralize method only works with english words. If you need language independent pluralization then have a look at the [I18n](:base_url:/docs/:version:/learn-more:internationalization) docs.

The ```camel2underscored``` method will convert camel case to underscored.

	$str = Str::camel2underscored('HelloWorld'); // $str will be set to "hello_world"

The ```underscored2camel``` method will convert underscored to camel case.

	$str = Str::underscored2camel('hello_world'); // $str will be set to "helloWorld"
	$str = Str::underscored2camel('hello_world', true); // $str will be set to "HelloWorld"