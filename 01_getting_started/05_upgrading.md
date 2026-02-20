# Upgrading

* [Controller helper](#controller_helper)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako 12.0.x to 12.1.x.

There are no breaking changes in this release but there are some deprecations. Check out the changelog to check out the new features included in the release. Follow this upgrade guide to future-proof your application.

--------------------------------------------------------

### <a id="controller_helper" href="#controller_helper">Controller helper</a>

The `fileResponse`, `streamResponse`, and `jsonResponse` methods of the `ControllerHelperTrait` (used by the optional Mako base controller) have been deprecated. The underlying functionality remains available, but you must now use the corresponding response classes directly.

| Deprecated method | Class                             |
|-------------------|-----------------------------------|
| fileResponse      | mako\http\response\senders\File   |
| streamResponse    | mako\http\response\senders\Stream |
| jsonResponse      | mako\http\response\builders\JSON  |

```
// Before

return $this->fileResponse('/files/file.ext');

// Now

return new File('/files/file.ext');
```

```
// Before

return $this->streamResponse(function ($stream) {
	$stream->flush('Hello, world!');

	sleep(2);

	$stream->flush('Hello, world!');
}, 'text/plain', 'UTF-8');

// Now

return new Stream(function ($stream) {
	$stream->flush('Hello, world!');

	sleep(2);

	$stream->flush('Hello, world!');
}, 'text/plain', 'UTF-8');
```

```
// Before

return $this->jsonResponse([1, 2, 3]);

// Now

return new JSON([1, 2, 3]);
```