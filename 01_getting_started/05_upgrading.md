# Upgrading
--------------------------------------------------------

* [13.0](#13.0)
    - [Dotenv loader](#13.0-dotenv-loader)
    - [Controller helper](#13.0-controller_helper)
    - [Enum cases](#13.0-enum_cases)
    - [Query builder sorting](#13.0-query_builder_sorting)
    - [HTTP statuses](#13.0-http_statuses)
    - [Uploaded files](#13.0-uploaded_files)
    - [Input validation](#13.0-input_validation)
    - [Date and time](#13.0-date_and_time)
    - [Image processing](#13.0-image-processing)
    - [Signer](#13.0-signer)

--------------------------------------------------------

### <a id="13.0" href="#13.0">13.0</a>

This portion of the guide takes you through the steps needed to migrate from Mako 12.* to 13.0.

While most applications won't be affected by every change, we recommend reviewing the entire guide to make sure your application is ready for 13.0.

#### <a id="13.0-dotenv-loader" href="#13.0-dotenv-loader">Dotenv loader</a>

In Mako 13, we added a `dotenv` loader for use in development environments. This allows you to define environment variables in a `.env` file located in the application root directory. Make sure to add the `.env` file to your `.gitignore` file (or equivalent ignore file for your VCS) to prevent sensitive values from being committed.

Add the following to the end of your `init.php` file to enable it:

```php
/*
 * Load .env if it exists.
 */
if (file_exists($dotenv = dirname(__DIR__) . '/.env')) {
	(new DotenvLoader)->load($dotenv);
}
```

#### <a id="13.0-controller_helper" href="#13.0-controller_helper">Controller helper</a>

The `fileResponse`, `streamResponse`, and `jsonResponse` methods of the `ControllerHelperTrait` (used by the optional Mako base controller) have been removed. The underlying functionality remains available, but you must now use the corresponding response classes directly.

| Deprecated method | Class                             |
|-------------------|-----------------------------------|
| fileResponse      | mako\http\response\senders\File   |
| streamResponse    | mako\http\response\senders\Stream |
| jsonResponse      | mako\http\response\builders\JSON  |

File streams:

```php{1}
{- return $this->fileResponse('/files/file.ext'); -}
{+ return new File('/files/file.ext'); +}
```

Custom stream responses:

```php
// Before

return $this->streamResponse(function ($stream) {
	$stream->flush('Hello, world!');

	sleep(2);

	$stream->flush('Hello, world!');
}, 'text/plain', 'UTF-8');

// Now

return new Stream(function () {
	yield 'Hello, world!';

	sleep(2);

	yield 'Hello, world!';
}, 'text/plain', 'UTF-8');
```

> Note that calls to the `Stream::flush()` method have been replaced by yielding.

JSON responses:

```php{1}
{- return $this->jsonResponse([1, 2, 3]); -}
{+ return new JSON([1, 2, 3]); +}
```

#### <a id="13.0-enum_cases" href="#13.0-enum_cases">Enum cases</a>

All enum cases have been converted from `UPPER_SNAKE_CASE` to `PascalCase` to follow the standard PHP convention for naming enum cases. The reason they were previously in `UPPER_SNAKE_CASE` is that many of them were converted from classes with class constants.

```php{1}
{- Status::NOT_FOUND -}
{+ Status::NotFound +}
```

Here's a list of the affected enums:

- mako\cli\input\Key
- mako\database\query\SortDirection
- mako\database\query\VectorDistance
- mako\env\Type
- mako\file\Permission
- mako\gatekeeper\LoginStatus
- mako\http\response\Status
- mako\http\response\senders\stream\event\Type
- mako\pixel\image\operations\AspectRatio
- mako\pixel\image\operations\Flip
- mako\pixel\image\operations\WatermarkPosition
- mako\pixel\metadata\xmp\properties\Type

#### <a id="13.0-query_builder_sorting" href="#13.0-query_builder_sorting">Query builder sorting</a>

The option to pass the string values `ASC` and `DESC` to the following methods has been removed in favor of the `SortDirection` enum:

- Query::orderBy()
- Query::orderByRaw()
- Query::orderByVectorDistance()

```php{1}
{- $query->orderBy('id', 'DESC') -}
{+ $query->orderBy('id', SortDirection::Descending) +}
```

The `$order` parameter of these methods has been renamed to `$sortDirection`. This is only relevant if you are using named parameters:

```php{1}
{- $query->orderBy('id', order: SortDirection::Descending) -}
{+ $query->orderBy('id', sortDirection: SortDirection::Descending) +}
```

You may still use the following methods without any changes:

- Query::ascending()
- Query::descending()
- Query::ascendingRaw()
- Query::descendingRaw()
- Query::ascendingVectorDistance()
- Query::descendingVectorDistance()

#### <a id="13.0-http_statuses" href="#13.0-http_statuses">HTTP statuses</a>

You can no longer use an integer to set the response status code. A `Status` enum case or `StatusInterface` implementation must be used instead.

```php{1}
{- $response->setStatus(404) -}
{+ $response->setStatus(Status::NotFound) +}
```

The following `Status` enum cases have been renamed to conform with the HTTP standard:

| Old name                    |  New name                    |
|-----------------------------|------------------------------|
| Status::PayloadTooLarge     | Status::ContentTooLarge      |
| Status::UnprocessableEntity | Status::UnprocessableContent |

The `Status::UseProxy` case has been deprecated per [⁠RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231#section-6.4.5).

The following non-standard `Status` enum cases have been removed:

- Status::InvalidToken
- Status::TokenRequired

If you still need to use these status codes, you can use the `CustomStatus` class instead:

```php{1}
{- $response->setStatus(Status::InvalidToken) -}
{+ $response->setStatus(new CustomStatus(498, 'Invalid Token')) +}
```

#### <a id="13.0-uploaded_files" href="#13.0-uploaded_files">Uploaded files</a>

The `UploadedFile::moveTo()` method now returns a `FileInfo` instance instead of a boolean, allowing you to retrieve information about the file after it has been moved. An `UploadException` is thrown in cases where the method previously returned `false`.

```php
try {
    $file = $uploadedFile->moveTo('/mnt/storage/file.ext');
    $size = $file->getSize();
}
catch(UploadException $e) {
    // Handle failure
}
```

#### <a id="13.0-input_validation" href="#13.0-input_validation">Input validation</a>

The `InputValidation` middleware now sets the response status code to `422` (unprocessable content) instead of `400` (bad request) when input validation fails.

You can restore the previous behavior by extending the middleware with a custom implementation:

```php
namespace app\http\middleware;

use mako\http\exceptions\BadRequestException;
use mako\http\response\Status;
use mako\validator\input\http\routing\middleware\InputValidation as BaseInputValidation;
use Override;

class InputValidation extends BaseInputValidation
{
    #[Override]
	protected const Status HTTP_STATUS = Status::BadRequest;

    #[Override]
	protected const string HTTP_STATUS_EXCEPTION = BadRequestException::class;
}
```

#### <a id="13.0-date_and_time" href="#13.0-date_and_time">Date and time</a>

Everywhere that previously returned a `Time` instance now returns a `TimeImmutable` instance. This should not cause any issues unless you type hint `Time` instead of `DateTimeInterface` or `TimeInterface`, which both classes implement.

If you previously mutated returned `Time` instances directly, update your code to assign the modified instance instead:

```php{1}
{- $time->modify('+1 day'); -}
{+ $time = $time->modify('+1 day'); +}
```

The `Time::getImmutable()` method has been renamed to `Time::toImmutable()`.

```php{1}
{- $immutable = $mutable->getImmutable(); -}
{+ $immutable = $mutable->toImmutable(); +}
```

The `TimeImmutable::getMutable()` method has been renamed to `TimeImmutable::toMutable()`.

```php{1}
{- $mutable = $immutable->getMutable(); -}
{+ $mutable = $immutable->toMutable(); +}
```

The `TimeInterface::copy()` method has been removed. The same functionality can be achieved with the `clone` keyword:

```php{1}
{- $copy = $time->copy(); -}
{+ $copy = clone $time; +}
```

#### <a id="13.0-image-processing" href="#13.0-image-processing">Image processing</a>

The `ImageInterface::getImageBlob()` method has been renamed to `ImageInterface::toBlob()`.

```php{1}
{- $blob = $image->getImageBlob(); -}
{+ $blob = $image->toBlob(); +}
```

The `ImageInterface::snapshot()` and `ImageInterface::restore()` methods have been removed. The same functionality can now be achieved using the `clone` keyword or using the `ImageInterface::applyOnClone()` method.

```php
// Before

$image = new ImageMagick('image.png');
$image->apply(new Resize(48, 48));
$image->snapshot();
$image->apply(new Flip());
$image->save('flipped.png');
$image->restore();
$image->save('image.png');

// Now using clone

$image = new ImageMagick('image.png');
$image->apply(new Resize(48, 48));
(clone $image)->apply(new Flip())->save('flipped.png');
$image->save('image.png');

// Now using applyOnClone

$image = new ImageMagick('image.png');
$image->apply(new Resize(48, 48));
$image->applyOnClone(new Flip())->save('flipped.png');
$image->save('image.png');
```

The `ImageInterface::getTopColors()` method has been removed. Use the new `TopColors` inspector instead.

```php{1}
{- $topColors = $image->getTopColors(); -}
{+ $topColors = $image->inspect(new TopColors); +}
```

The undocumented percentage-based scaling behavior when only passing a single parameter to the `Resize` operation has been removed. The `Resize` operation now always works with explicit dimensions. Use the new `Scale` operation when you need to scale an image by a percentage.

```php{1}
{- $image->apply(new Resize(50)); -}
{+ $image->apply(new Scale(50)); +}
```

#### <a id="13.0-signer" href="#13.0-signer">Signer</a>

The `Signer` class has been moved from the `mako\security` namespace to  the `mako\security\signer` namespace.

```php{1}
{- use mako\security\Signer; -}
{+ use mako\security\signer\Signer; +}
```
