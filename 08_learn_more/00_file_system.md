# File system

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The ```FileSystem``` class contains methods that assist in working with the file system.

--------------------------------------------------------

<a id="usage"></a>

### Usage

You can create a new FileSystem object or fetch the instance present in the [IoC container](:base_url:/docs/:version:/getting-started:dependency-injection). In the following examples we'll assume that you'll using the instance from the container.

The ```has``` method return TRUE if the provided path exists and FALSE if not.

	$exists = $this->fileSystem->has('/foo/bar.txt');

The ```isFile``` method return TRUE if the provided path is a file and FALSE if not.

	$isFile = $this->fileSystem->isFile('/foo/bar.txt');

The ```isDirectory``` method returns TRUE if the provided path is a directory and FALSE if not.

	$isDirectory = $this->fileSystem->isDirectory('/foo');

The ```isEmpty``` method returns TRUE if the provided path is an empty file or directory and FALSE if not.

	$isEmpty = $this->fileSystem->isEmpty('/foo');

The ```isReadable``` method returns TRUE if the provided path is readable and FALSE if not.

	$isReadable = $this->fileSystem->isReadable('/foo/bar.txt');

The ```isWritable``` method returns TRUE if the provided path is writable and FALSE if not.

	$isWritable = $this->fileSystem->isWritable('/foo/bar.txt');

The ```lastModified``` method returs the time (unix timestamp) when the data blocks of a file were being written to, that is, the time when the content of the file was changed.

	$lastModified = $this->fileSystem->lastModified('/foo/bar.txt');

The ```size``` method returns the size of the file in bytes.

	$size = $this->fileSystem->size('/foo/bar.txt');

The ```extension``` method returns the extension of the file.

	$extension = $this->fileSystem->extension('/foo/bar.txt');

The ```mime``` method returns the mime type of the file.  It returns FALSE if the mime type is not found.

	$mime = $this->fileSystem->mime('/foo/bar.txt');

> The method will try to guess the mimetype by using the file extension if the [finfo_open()](http://php.net/manual/en/function.finfo-open.php) function doesn't exist. Note that this is not a very reliable way of determinating a mime type. You can disable guessing by setting the second parameter to FALSE.

The ```remove``` method will delete a file from disk.

	$this->fileSystem->remove('/foo/bar.txt');

The ```glob``` method returns an array of path names matching the provided pattern.

	$paths = $this->fileSystem->glob('/foo/*.txt');

The ```get``` method returns the contents of a file.

	$contents = $this->fileSystem->getContents('/foo/bar.txt');

The ```put``` method puts the provided contents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->put('/foo/bar.txt', 'hello, world!');

The ```prepend``` method will prepend the provided conetents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->prepend('/foo/bar.txt', 'hello, world!');

The ```appendContents``` method will append the provided conetents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->append('/foo/bar.txt', 'hello, world!');

The ```truncate``` method will truncate the contents of the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->truncate('/foo/bar.txt');

The ```include``` method will include a file.

	$this->fileSystem->include('/foo/bar.txt');

The ```includeOnce``` method will include a file if it hasn't already been included.

	$this->fileSystem->includeOnce('/foo/bar.txt');

The ```require``` method will require a file.

	$this->fileSystem->require('/foo/bar.txt');

The ```requireOnce``` method will require a file if it hasn't already been required.

	$this->fileSystem->requireOnce('/foo/bar.txt');

The ```hash``` method generates a hash value using the contents of the given file. The default hashing algorithm is `sha256` but you can override it using the optional second parameter.

	$hash = $this->fileSystem->hash('/foo/bar.txt');

The ```hmac``` method a keyed hash value using the HMAC method using the contents of the given file. The default hashing algorithm is `sha256` but you can override it using the optional third parameter.

	$hash = $this->fileSystem->hmac('/foo/bar.txt', $secret);

The ```file``` method will return a [SplFileObject](http://php.net/manual/en/class.splfileobject.php).

	$file = $this->fileSystem->file('/foo/bar.txt', 'r');