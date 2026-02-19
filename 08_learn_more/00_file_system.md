# File system

--------------------------------------------------------

* [File system](#file_system)
* [Permissions](#permissions)
    - [Permission enum](#permissions:permission_enum)
    - [Permission collection](#permissions:permission_collection)
* [File info](#file_info)
* [Finder](#finder)

--------------------------------------------------------

The file library contains classes that assist you with working with the files and the file system.

--------------------------------------------------------

### <a id="file_system" href="#file_system">File system</a>

You can create a new `FileSystem` object or fetch the instance present in the [dependency injection container](:base_url:/docs/:version:/getting-started:dependency-injection). In the following examples we'll assume that you'll using the instance from the container.

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

The `setPermissions` method allows you to set the file permissions. You can pass an integer representing the permissions or a [`Permissions`](#permissions:permission_collection) instance.

```
// Set the permissions to 644 using an octal

$this->fileSystem->setPermissions('/foo/bar.txt', 0o644);

// Set the permissions to 644 using a Permissions instance

$this->fileSystem->setPermissions('/foo/bar.txt', new Permissions(
    Permission::OWNER_WRITE_READ,
    Permission::GROUP_READ,
    Permission::PUBLIC_READ,
));
```

The `getPermissions` method returns a [`Permissions`](#permissions:permission_collection) instance representing the permissions of the given file or directory.

```
$permissions = $this->fileSystem->getPermissions('/foo/bar.txt');

if($permissions->hasPermissions(Permission::PUBLIC_READ)) {
    // The file is readable for everyone
}
```

The `isReadable` method returns `true` if the provided path is readable and `false` if not.

```
$isReadable = $this->fileSystem->isReadable('/foo/bar.txt');
```

The `isWritable` method returns `true` if the provided path is writable and `false` if not.

```
$isWritable = $this->fileSystem->isWritable('/foo/bar.txt');
```

The `isExecutable` method returns `true` if the provided path is executable and `false` if not.

```
$isWritable = $this->fileSystem->isExecutable('/foo/bar.txt');
```

The `isLink` method returns `true` if the provided path is a link and `false` if not.

```
$isLink = $this->fileSystem->isLink('/foo/bar.txt');
```

The `getLinkTarget` method returns the path to the link target.

```
$linkTarget = $this->fileSystem->getLinkTarget('/foo/bar.txt');
```

The `createSymbolicLink` creates a symbolic link to the chosen target.

```
$this->fileSystem->createSymbolicLink('/foo/bar.txt', '/foo/baz.txt');
```

The `createHardLink` creates a hard link to the chosen target.

```
$this->fileSystem->createHardLink('/foo/bar.txt', '/foo/baz.txt');
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

The `copy` method will copy a file from the chosen destination.

```
$this->fileSystem->copy('/foo/bar.txt', '/bar/foo.txt');
```

The `rename` method will rename (move) a file.

```
$this->fileSystem->rename('/foo/bar.txt', '/bar/foo.txt');
```

The `remove` method will delete a file from disk.

```
$this->fileSystem->remove('/foo/bar.txt');
```

The `removeDirectory` method will delete a directory and all its contents from disk.

```
$this->fileSystem->removeDirectory('/foo');
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

The `createDirectory` creates a directory.

```
$this->fileSystem->createDirectory('/foo');
```

The `getDiskSize` method returns the total size of a filesystem or disk partition in bytes.

```
$diskSize = $this->fileSystem->getDiskSize();

$diskSize = $this->fileSystem->getDiskSize('/mnt/filesystem');
```

The `getFreeSpaceOnDisk` method returns the total number of available bytes on the filesystem or disk partition.

```
$freeSpace = $this->fileSystem->getFreeSpaceOnDisk();

$diskSize = $this->fileSystem->getFreeSpaceOnDisk('/mnt/filesystem');
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

The `file` method will return a [SplFileObject](https://php.net/manual/en/class.splfileobject.php) instance.

```
$file = $this->fileSystem->file('/foo/bar.txt', 'r');
```

The `clearCache` will clear the [status cache](https://www.php.net/manual/en/function.clearstatcache.php).

```
$this->fileSystem->clearCache();
```

--------------------------------------------------------

### <a id="permissions" href="#permissions">Permissions</a>

#### <a id="permissions:permission_enum" href="#permissions:permission_enum">Permission enum</a>

The `Permission` enum contains cases that represent all the different owner, group, and public permissions, as well as the special bits.

| Case                                     | Permission in octal form |
|------------------------------------------|--------------------------|
| Permission::NONE                         | 0o0000                    |
| Permission::OWNER_EXECUTE                | 0o0100                    |
| Permission::OWNER_WRITE                  | 0o0200                    |
| Permission::OWNER_EXECUTE_WRITE          | 0o0300                    |
| Permission::OWNER_READ                   | 0o0400                    |
| Permission::OWNER_EXECUTE_READ           | 0o0500                    |
| Permission::OWNER_WRITE_READ             | 0o0600                    |
| Permission::OWNER_FULL                   | 0o0700                    |
| Permission::GROUP_EXECUTE                | 0o0010                    |
| Permission::GROUP_WRITE                  | 0o0020                    |
| Permission::GROUP_EXECUTE_WRITE          | 0o0030                    |
| Permission::GROUP_READ                   | 0o0040                    |
| Permission::GROUP_EXECUTE_READ           | 0o0050                    |
| Permission::GROUP_WRITE_READ             | 0o0060                    |
| Permission::GROUP_FULL                   | 0o0070                    |
| Permission::PUBLIC_EXECUTE               | 0o0001                    |
| Permission::PUBLIC_WRITE                 | 0o0002                    |
| Permission::PUBLIC_EXECUTE_WRITE         | 0o0003                    |
| Permission::PUBLIC_READ                  | 0o0004                    |
| Permission::PUBLIC_EXECUTE_READ          | 0o0005                    |
| Permission::PUBLIC_WRITE_READ            | 0o0006                    |
| Permission::PUBLIC_FULL                  | 0o0007                    |
| Permission::FULL                         | 0o0777                    |
| Permission::SPECIAL_STICKY               | 0o1000                    |
| Permission::SPECIAL_SETGID               | 0o2000                    |
| Permission::SPECIAL_SETUID               | 0o4000                    |

The `calculate` method returns an integer representing the sum of the chosen permissions.

```
$permissions = Permission::calculate(
    Permission::OWNER_WRITE_READ,
    Permission::GROUP_READ,
    Permission::PUBLIC_READ,
);
```

The `hasPermissions` method allows you to check if a permission set contains the chosen permission(s).

```
$hasPublicRead = Permission::hasPermissions(0o644, Permission::PUBLIC_READ);

$hasGroupAndPublicRead = Permission::hasPermissions(0o644, Permission::GROUP_READ, Permission::PUBLIC_READ);
```

#### <a id="permissions:permission_collection" href="#permissions:permission_collection">Permission collection</a>

The `Permissions` class allows you to work with a collection of `Permission` instances. You can create a collection using the constructor or by using the `fromInt` method.

```
// Create an instance using the constructor

$permissions = new Permissions(
    Permission::OWNER_WRITE_READ,
    Permission::GROUP_READ,
    Permission::PUBLIC_READ,
);

// Create an instance from an integer

$permissions = Permissions::fromInt(0o644);
```

The `add` method adds one or more `Permission` instances to the collection.

```
$permissions->add(Permission::GROUP_WRITE);
```

The `remove` method removes one or more `Permission` instances from the collection.

```
$permissions->remove(Permission::GROUP_WRITE);
```

The `getPermissions` method will return an array of `Permission` instances.

```
$permissions = $permissions->getPermissions();
```

The `hasPermissions` method allows you to check if the permission collection contains the chosen permission(s).

```
$hasPublicRead = $permissions->hasPermissions(Permission::PUBLIC_READ);

$hasGroupAndPublicRead = $permissions->hasPermissions(Permission::GROUP_READ, Permission::PUBLIC_READ);
```

The `toInt` returns an integer representation of the permissions.

```
$permissions = $permissions->toInt(); // 420 (0o644)
```

The `toOctalString` returns an octal string representation of the permissions.

```
$permissions = $permissions->toOctalString(); // "644"
```

The `toRwxString` method returns a <abbr title="Read Write Execute">RWX</abbr> representation of the permissions.

```
$permissions = $permissions->toRwxString(); // rw-r--r--
```

--------------------------------------------------------

### <a id="file_info" href="#file_info">File info</a>

You can create a new `FileInfo` instance using the constructor or by getting an instance via the `FileSystem::info()` method. The `FileInfo` class extends the [`SplFileInfo`](https://php.net/manual/en/class.splfileinfo.php) class with a set of useful methods that we'll document below.

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

> You should provide a cryptographically secure key when generating the hash. You can easily generate your own secure keys using the `app:generate-key` reactor command.
{.danger}

Validating files against the keyed hash is done using the `validateHmac` method. If you didn't use the default `sha256` algorithm when generating the hash then you'll have to specify which one you used using the optional third parameter. If the hash you're comparing against is provided in its raw binary form then you'll have to set the  optional fourth parameter to `true`.

```
$valid = $info->validateHmac($hash, $key);
```

The `getPermissions` method returns a [`Permissions`](#permissions:permission_collection) instance that allows you to get information about the permissions of the file.

```
$permissions = $info->getPermissions();

$rwx = $permissions->toRwxString(); // rw-r--r--
```

--------------------------------------------------------

### <a id="finder" href="#finder">Finder</a>

The finder class allows you to find files in the filesystem.

```
$finder = new Finder(['/files']);

foreach ($finder->find() as $file) {
    echo $file;
}
```

The `find` and `findAs` methods return generators so if you want to collect all files into an array then you can use the `iterator_to_array` function.

```
$files = iterator_to_array($finder->find());
```

By default the finder will search recursively down in the directory tree. You can limit the depth of the search by using the `setMaxDepth` method.

```
$finder->setMaxDepth(2);
```

You can also filter files using a regex pattern.

```
$finder->setPattern('/\.php$/'); // Only match files with a .php extension
```

In addition to the `find` method you can also use the `findAs` method that alows you iterate over objects instead of just strings containg the file path.

```
foreach ($finder->findAs(FileInfo::class) as $fileInfo) {
    echo $fileInfo->getMimeType();
}
```