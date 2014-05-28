# File system

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The ```FileSystem``` class contains methods that assist in working with the file system.

--------------------------------------------------------

<a id="usage"></a>

### Usage

You can create a new FileSystem object or fetch the instance present in the [IoC container](:base_url:/docs/:version:/getting-started:dependency-injection). In the following examples we'll assume that you'll using the instance from the container.

The ```exists``` method return TRUE if the provided path exists and FALSE if not.

	$exists = $this->fileSystem->exists('/foo/bar.txt');

The ```isFile``` method return TRUE if the provided path is a file and FALSE if not.

	$isFile = $this->fileSystem->isFile('/foo/bar.txt');

The ```isDirectory``` method returns TRUE if the provided path is a directory and FALSE it not.

	$isDirectory = $this->fileSystem->isDirectory('/foo');

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

The ```delete``` method will delete a file from disk.

	$this->fileSystem->delete('/foo/bar.txt');

The ```glob``` method returns an array of path names matching the provided pattern.

	$paths = $this->fileSystem->glob('/foo/*.txt');

The ```getContents``` method returns the contents of a file.

	$contents = $this->fileSystem->getContents('/foo/bar.txt');

The ```putContents``` method puts the provided contents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->putContents('/foo/bar.txt', 'hello, world!');

The ```prependContents``` method will prepend the provided conetents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->prependContents('/foo/bar.txt', 'hello, world!');

The ```appendContents``` method will append the provided conetents to the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->appendContents('/foo/bar.txt', 'hello, world!');

The ```truncateContents``` method will truncate the contents of the file. There's an optional third parameter that will set an exclusive write lock if set to TRUE.

	$this->fileSystem->truncateContents('/foo/bar.txt');

The ```includeFile``` method will include a file.

	$this->fileSystem->includeFile('/foo/bar.txt');

The ```includeFileOnce``` method will include a file if it hasn't already been included.

	$this->fileSystem->includeFileOnce('/foo/bar.txt');

The ```requireFile``` method will require a file.

	$this->fileSystem->requireFile('/foo/bar.txt');

The ```requireFileOnce``` method will require a file if it hasn't already been required.

	$this->fileSystem->requireFileOnce('/foo/bar.txt');

The ```file``` method will return a [SplFileObject](http://php.net/manual/en/class.splfileobject.php).

	$file = $this->fileSystem->file('/foo/bar.txt', 'r');