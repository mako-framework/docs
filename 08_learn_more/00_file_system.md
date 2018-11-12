# File system

--------------------------------------------------------

* [File system](#file_system)
* [File info](#file_info)

--------------------------------------------------------

The file library contains classes that assist you with working with the files and the file system.

--------------------------------------------------------

<a id="file_system"></a>

### File system

You can create a new `FileSystem` object or fetch the instance present in the [IoC container](:base_url:/docs/:version:/getting-started:dependency-injection). In the following examples we'll assume that you'll using the instance from the container.

The `has` method returns `true` if the provided path exists and `false` if not.

```
$exists = $this->fileSystem->has('/foo/bar.txt');
```

The `isFile` method returns `true` if the provided path is a file and `false` if not.

```
$isFile = $this->fileSystem->isFile('/foo/bar.txt');
```

The `isDirectory` method returns `true` if the provided path is a directory and `false` if not.

```
$isDirectory = $this->fileSystem->isDirectory('/foo');
```

The `isEmpty` method returns `true` if the provided path is an empty file or directory and `false` if not.

```
$isEmpty = $this->fileSystem->isEmpty('/foo');
```

The `isReadable` method returns `true` if the provided path is readable and `false` if not.

```
$isReadable = $this->fileSystem->isReadable('/foo/bar.txt');
```

The `isWritable` method returns `true` if the provided path is writable and `false` if not.

```
$isWritable = $this->fileSystem->isWritable('/foo/bar.txt');
```

The `lastModified` method returns the time (unix timestamp) when the data blocks of a file were being written to, that is, the time when the content of the file was changed.

```
$lastModified = $this->fileSystem->lastModified('/foo/bar.txt');
```

The `size` method returns the size of the file in bytes.

```
$size = $this->fileSystem->size('/foo/bar.txt');
```

The `extension` method returns the extension of the file.

```
$extension = $this->fileSystem->extension('/foo/bar.txt');
```

The `remove` method will delete a file from disk.

```
$this->fileSystem->remove('/foo/bar.txt');
```

The `glob` method returns an array of path names matching the provided pattern.

```
$paths = $this->fileSystem->glob('/foo/*.txt');
```

The `get` method returns the contents of a file.

```
$contents = $this->fileSystem->get('/foo/bar.txt');
```

The `put` method puts the provided contents to the file. There's an optional third parameter that will set an exclusive write lock if set to `true`.

```
$this->fileSystem->put('/foo/bar.txt', 'hello, world!');
```

The `prepend` method will prepend the provided contents to the file. There's an optional third parameter that will set an exclusive write lock if set to `true`.

```
$this->fileSystem->prepend('/foo/bar.txt', 'hello, world!');
```

The `appendContents` method will append the provided contents to the file. There's an optional third parameter that will set an exclusive write lock if set to `true`.

```
$this->fileSystem->append('/foo/bar.txt', 'hello, world!');
```

The `truncate` method will truncate the contents of the file. There's an optional second parameter that will set an exclusive write lock if set to `true`.

```
$this->fileSystem->truncate('/foo/bar.txt');
```

The `include` method will include a file.

```
$this->fileSystem->include('/foo/bar.txt');
```

The `includeOnce` method will include a file if it hasn't already been included.

```
$this->fileSystem->includeOnce('/foo/bar.txt');
```

The `require` method will require a file.

```
$this->fileSystem->require('/foo/bar.txt');
```

The `requireOnce` method will require a file if it hasn't already been required.

```
$this->fileSystem->requireOnce('/foo/bar.txt');
```

The `info` method will return a `FileInfo` instance.

```
$info = $this->fileSystem->info('/foo/bar.txt');
```

The `file` method will return a [SplFileObject](http://php.net/manual/en/class.splfileobject.php) instance.

```
$file = $this->fileSystem->file('/foo/bar.txt', 'r');
```

--------------------------------------------------------

<a id="file_info"></a>

### File info

You can create a new `FileInfo` instance using the constructor or by getting an instance via the `FileSystem::info()` method. The `FileInfo` class extends the [`SplFileInfo`](http://php.net/manual/en/class.splfileinfo.php) class with a set of useful methods that we'll document below.

```
$info = new FileInfo('/foo/bar.txt');
```

The `getMimeType` method will return the MIME type of the file.

```
$mime = $info->getMimeType();
```

The `getMimeEncoding` method will return the MIME encoding of the file.

```
$encoding = $info->getMimeEncoding();
```

You can generate a hash based on the contents of the file using the `getHash` method. The default algorithm is `sha256` but you can specify which one you would like to use using the optional first parameter. You can also tell the method to return the hash in its raw binary form by setting the optional second parameter to `true`.

```
$hash = $info->getHash();
```

You can validate a file against a known hash using the `validateHash` method. If you didn't use the default `sha256` algorithm when generating the hash then you'll have to specify which one you used using the optional second parameter. If the hash you're comparing against is provided in its raw binary form then you'll have to set the  optional third parameter to `true`.

```
$valid = $info->validateHash($hash);
```
A keyed hash can be generated using the `getHmac` method. The default algorithm is `sha256` but you can specify which one you would like to use using the optional second parameter. You can also tell the method to return the hash in its raw binary form by setting the optional third parameter to `true`.

```
$hash = $info->getHmac($key);
```

> You should provide a cryptographically secure key when generating the hash. You can easily generate your own secure keys using the `app.generate_key` reactor command.
{.danger}

Validating files against the keyed hash is done using the `validateHmac` method. If you didn't use the default `sha256` algorithm when generating the hash then you'll have to specify which one you used using the optional third parameter. If the hash you're comparing against is provided in its raw binary form then you'll have to set the  optional fourth parameter to `true`.

```
$valid = $info->validateHmac($hash, $key);
```
