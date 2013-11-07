# File helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The ```File``` class contains methods that assist in working with files.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```size``` method returns a human friendly representation of the file size.

	echo File::size(1024); // Will print "1 KiB"

	echo File::size(1024, false); // Will print "1.02 KB"

	echo File::size('/path/to/file.ext'); // Will print "2.3 MiB"

The ```mime``` method returns the mime type of a file. It returns FALSE if the mime type is not found.

The method will try to guess the mimetype by using the file extension if the [finfo_open()](http://php.net/manual/en/function.finfo-open.php) function doesn't exist. Note that this is not a very reliable way of determinating a mime type. You can disable guessing by setting the second parameter to FALSE.

	$type = File::mime('/path/to/my_file.png'); // Should return "image/png"

The ```display``` method will render the file in the browser.

	// The file will be displayed in the browser and the content type will be detected automatically

	File::display('/path/to/my_file.png');

The ```download``` method forces your browser to download the file.

	// The file will be downloaded as 'my_file.png' and the content type will be detected automatically

	File::download('/path/to/my_file.png');

	// The file will be downloaded as 'my_file.png'

	File::download('/path/to/my_file.png', 'image/png');

	// The file will be downloaded as 'your_file.png'

	File::download('/path/to/my_file.png', 'image/png', 'your_file.png');

	// The file will be downloaded as 'my_file.png' and the maximum download speed is set to 5 KiB/s

	File::download('/path/to/my_file.png', null, null, 5);

The ```split``` method will split a big file into smaller files.

	// Will split the 100 MiB file into five 20 MiB files.

	File::split('/path/to/my/100MiB-file.ext', 20);

The ```merge``` method will merge files that have been split.

	// Will split the five 20 MiB files into a 100 MiB file.

	File::merge('/path/to/my/100MiB-file.ext');