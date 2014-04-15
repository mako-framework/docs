# File helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The ```File``` class contains methods that assist in working with files.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```size``` method returns returns the filesize in bytes.

	$size = File::size('example.txt');

The ```mime``` method returns the mime type of a file. It returns FALSE if the mime type is not found.

	$type = File::mime('image.png'); // Should return "image/png"

> The method will try to guess the mimetype by using the file extension if the [finfo_open()](http://php.net/manual/en/function.finfo-open.php) function doesn't exist. Note that this is not a very reliable way of determinating a mime type. You can disable guessing by setting the second parameter to FALSE.

The ```get``` method returns the file contents.

	$content = File::get('example.txt');

The ```put``` method writes the supplied data to the file. The file will be created if it doesn't exist.

	File::put('example.txt', 'Hello, world!');

	// Set the third parameter to TRUE to acquire an exclusive write lock

	File::put('example.txt', 'Hello, world!', true);

> The method will overide any existing file content.

The ```prepend``` method will prepend the supplied data to a file.

	File::prepend('example.txt', 'Hello, world!');

	// Set the third parameter to TRUE to acquire an exclusive write lock

	File::prepend('example.txt', 'Hello, world!', true);

The ```append``` method will prepend the supplied data to a file.

	File::append('example.txt', 'Hello, world!');

	// Set the third parameter to TRUE to acquire an exclusive write lock

	File::append('example.txt', 'Hello, world!', true);

The ```truncate``` method will clear the contents of a file.

	File::truncate('example.txt');

	// Set the second parameter to TRUE to acquire an exclusive write lock

	File::truncate('example.txt', true);