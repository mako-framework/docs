# Image processing

--------------------------------------------------------

* [Usage](#usage)
    - [Image information](#usage:image_information)
        - [Inspectors](#usage:image_information:inspectors)
    - [Image manipulation](#usage:image_manipulation)
    - [Image metadata](#usage:image_metadata)
        - [XMP](#usage:image_metadata:xmp)
            - [Reading](#usage:image_metadata:xmp:reading)
    - [Value objects](#usage:value_objects)
        - [Color](#usage:value_objects:color)
        - [Dimensions](#usage:value_objects:dimensions)
        - [Point](#usage:value_objects:point)
        - [Points](#usage:value_objects:points)

--------------------------------------------------------

The pixel library allows you to manipulate images through a consistent API using either GD or ImageMagick. GD generally provides faster processing for many operations and is available in most PHP installations, while ImageMagick offers more advanced image processing capabilities, improved color handling, and better support for complex formats such as animated GIFs and multi-frame images.

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

First you'll have to decide whether to use GD or ImageMagick. In this example we'll use the ImageMagick processor.

```php
$image = new ImageMagick('image.png');

// You can also create an instace from a path like this

$image = ImageMagick::fromPath('image.png');

// Or directly from a binary blob like this

$image = ImageMagick::fromBlob($blob);

// And finally from a stream

$image = ImageMagick::fromStream($stream);
```

#### <a id="usage:image_information" href="#usage:image_information">Image information</a>

The `getMimeType` method returns the mime type of your image.

```php
$mimeType = $image->getMimeType();
```

The `getHeight` method returns the height of your image.

```php
$height = $image->getHeight();
```

The `getWidth` method returns the width of your image.

```php
$width = $image->getWidth();
```

The `getDimensions` method returns a [`Dimensions`](#usage:value_objects:dimensions) instance representing the dimensions of the image.

```php
$dimensions = $image->getDimensions();

$width = $dimensions->width;
$height = $dimensions->height;
```

##### <a id="usage:image_information:inspectors" href="#usage:image_information:inspectors">Inspectors</a>

To retrieve more advanced image information you can use inspectors.

The `TopColors` inspector returns an array of the top `n` colors in an image, represented as [`Color`](#usage:value_objects:color) class instances. By default, it returns the top five colors, but you can specify a custom number of colors to return.

```php
$topColors = $image->inspect(new TopColors);

foreach ($topColors as $color) {
    var_dump($color->toHexString());
}
```

Here are all of the included image inspectors:

| Class                | Description                                                    | Gd | ImageMagick |
|----------------------|----------------------------------------------------------------|----|-------------|
| TopColors            | Returns the top colors of the image                            | ✓  | ✓           |

You can also create your own custom image inspectors by implementing the `InspectorInterface`.

```php
/**
 * @implements InspectorInterface<Dimensions>
 */
class Dimensions implements InspectorInterface
{
	/**
	 * {@inheritDoc}
	 */
	#[Override]
	public function inspect(object &$imageResource): Dimensions
	{
		return new Dimensions(
            $imageResource->getImageWidth(),
			$imageResource->getImageHeight()
        );
	}
}
```

> The generics annotation connects the inspector implementation with its return type. This allows your IDE and static analysis tools such as PHPStan to understand the value returned by `inspect()` and provide accurate type hints, auto-completion, and type checking.

#### <a id="usage:image_manipulation" href="#usage:image_manipulation">Image manipulation</a>

The `apply` method allows you to apply an image operation to your image.

```php
$image->apply(new Sharpen);
$image->apply(new Border(new Color(0, 0, 0, 127), width: 10));
```

You can also pipeline multiple operations using the `Pipeline` class.

```php
$image->apply(new Pipeline(
    new Sharpen,
    new Border(new Color(0, 0, 0, 127), width: 10),
));
```

The `applyOnClone` method allows you to apply an image operation to a copy of the image instance while leaving the original unchanged. Just like with the `apply` method, you can apply individual operations directly or use the `Pipeline` class to apply multiple operations.

```php
$greyscaleThumbnail = $image->applyOnClone(new Pipeline(
    new Resize(new Dimensions(300, 300)),
    new Greyscale,
));
```

Here are all of the included image operations:

| Class                | Description                                                    | Gd | ImageMagick |
|----------------------|----------------------------------------------------------------|----|-------------|
| Bezier               | Draws Bézier curve on the image                                | ✗  | ✓           |
| Bitonal              | Turns the image into a bitonal image                           | ✓  | ✓           |
| Border               | Draws a border around the image                                | ✓  | ✓           |
| Brightness           | Adjusts the brightness of the image (-100 to 100)              | ✓  | ✓           |
| Colorize             | Colorizes the image with the chosen color                      | ✓  | ✓           |
| Contrast             | Adjusts the contrast of the image (-100 to 100)                | ✓  | ✓           |
| Crop                 | Crops the image to the selected region                         | ✓  | ✓           |
| Flip                 | Flips the image                                                | ✓  | ✓           |
| Greyscale            | Turns the image into a greyscale image                         | ✓  | ✓           |
| Negate               | Negates the image                                              | ✓  | ✓           |
| Pixelate             | Pixelates the image                                            | ✓  | ✓           |
| Polygon              | Draws a polygon on the image                                   | ✓  | ✓           |
| Polyline             | Draws a polyline on the image                                  | ✓  | ✓           |
| Resize               | Resizes the image to the chosen size                           | ✓  | ✓           |
| Rotate               | Rotates the image                                              | ✓  | ✓           |
| Saturation           | Adjusts the saturation of the image (-100 to 100)              | ✓  | ✓           |
| Scale                | Scales the image to the chosen percentage                      | ✓  | ✓           |
| Sepia                | Applies a sepia filter to the image                            | ✓  | ✓           |
| Sharpen              | Sharpens the image                                             | ✓  | ✓           |
| Temperature          | Adjusts the color temperature of the image (-100 to 100)       | ✓  | ✓           |
| Watermark            | Applies a custom watermark to the image                        | ✓  | ✓           |

You can also create your own custom image operations by implementing the `OperationInterface`.

```php
class MyOperation implements OperationInterface
{
    #[Override]
	public function apply(object &$imageResource): void
	{
        // Do your custom image operations here
    }
}
```

The `toBlob` method returns the raw binary image data.

```php
$image = $image->toBlob();

// You can also tell it to return a different image type

$image = $image->toBlob('jpg');

// You can also adjust the image quality in percent (default is 95%)

$image = $image->toBlob('jpg', 70);
```

The `toBase64` method returns a base64 encoded representation of the raw binary image data.

```php
$image = $image->toBase64();

// You can also tell it to return a different image type

$image = $image->toBase64('jpg');

// You can also adjust the image quality in percent (default is 95%)

$image = $image->toBase64('jpg', 70);
```

The `toDataUri` method returns a data uri representation of the image.

```php
$image = $image->toDataUri();

// You can also tell it to return a different image type

$image = $image->toDataUri('jpg');

// You can also adjust the image quality in percent (default is 95%)

$image = $image->toDataUri('jpg', 70);

// Display the image

echo "<image src='{$image}'>"; // <image src='data:image/jpeg;base64,...'>
```

The `toStream` method returns the image as a stream. By default, it uses `StreamStorage::Temp`, which creates a PHP `php://temp` stream. This stream stores data in memory until it reaches 2 MB before automatically spilling over to a temporary file on disk, providing a good balance between performance and memory usage. If you prefer to keep the entire stream in memory, pass `StreamStorage::Memory`, which creates a `php://memory` stream.

```php
$stream = $image->toStream();

// You can also tell it to return a different image type

$stream = $image->toStream('jpg');

// You can also adjust the image quality in percent (default is 95%)

$stream = $image->toStream('jpg', 70);

// And as mentioned above you can choose which stream wrapper to use

$stream = $image->toStream(stream: StreamStorage::Memory);
```

As the name suggests the `save` method will save your edited image to disk.

```php
// Override original file

$image->save();

// Create a new file

$image->save('edited_image.png');

// You can also adjust the image quality in percent (default is 95%)

$image->save('edited_image.png', 70);
```

#### <a id="usage:image_metadata" href="#usage:image_metadata">Image metadata</a>

The library also includes functionality for working with embedded image metadata.

##### <a id="usage:image_metadata:xmp" href="#usage:image_metadata:xmp">XMP</a>

###### <a id="usage:image_metadata:xmp:reading" href="#usage:image_metadata:xmp:reading">Reading</a>

The `XmpReader` class allows you to extract XMP metadata from the image file.

> Note: The XMP reader relies on [`FFI`](https://www.php.net/manual/en/book.ffi.php) and `libexempi`. The reader will attempt to auto-detect the shared library, but depending on your setup, you may need to specify it manually.

```php
$reader = new XmpReader('image.tif');

// You can specify the shared library if needed

$reader = new XmpReader('image.tif', 'libexempi.so.8');
```

The `toXml` method returns all the XMP data as XML.

```php
$xml = $reader->toXml();

// You can also cast the reader object to a string

$xml = (string) $reader;
```

The `getProperties` method returns all the XMP properties as objects.

```php
$properties = $reader->getProperties();

// You can also select the properties from a specific namespace

$properties = $reader->getProperties('http://purl.org/dc/elements/1.1/');
```

The `getProperty` method allows you to get a specific property. The first parameter is the property namespace and the second is the property name.

```php
$property = $reader->getProperty('http://purl.org/dc/elements/1.1/', 'title');

echo $property->value;
```

#### <a id="usage:value_objects" href="#usage:value_objects">Value objects</a>

##### <a id="usage:value_objects:color" href="#usage:value_objects:color">Color</a>

The `Color` class is a value object for representing colors. It is used by image [operations](#usage:image_manipulation) and [inspectors](#usage:image_information:inspectors).

```php
$color = new Color(0, 0, 0, 127);

// The color values can be accessed as read-only attributes

$red = $color->red;
$green = $color->green;
$blue = $color->blue;
$alpha = $color->alpha;
```

The following methods are available:

| Method                                         | Description                                                                           |
|------------------------------------------------|---------------------------------------------------------------------------------------|
| fromHex($hex)                                  | Creates a new Color instance from a hex value (e.g. "#FF0000")                      |
| getRed()                                       | Returns the red value (0-255)                                                         |
| getGreen()                                     | Returns the green value (0-255)                                                       |
| getBlue()                                      | Returns the blue value (0-255)                                                        |
| getAlpha()                                     | Returns the alpha value (0-255)                                                       |
| toHexString()                                  | Returns a hex string representation of the color (e.g. "#FF0000")                   |
| toHexaString()                                 | Returns a hexa string representation of the color (e.g. "#FF000000")                |
| toRgbString()                                  | Returns a rgb string representation of the color (e.g. "rgb(255, 0, 0)")            |
| toRgbaString()                                 | Returns a rgb string representation of the color (e.g. "rgba(255, 0, 0, 0.5)")      |
| toHslString()                                  | Returns a hsl string representation of the color (e.g. "hsl(0, 100.0%, 50.0%)")     |
| toHslaString()                                 | Returns a hsla string representation of the color (e.g. "hsl(0, 100.0%, 50.0%, 0.5)") |
| toHwbString()                                  | Returns a hwb string representation of the color (e.g. "hwb(0 0.0% 0.0%)")            |
| toHwbaString()                                 | Returns a hwba string representation of the color (e.g. "hwb(0 0.0% 0.0% / 0.5)")     |

##### <a id="usage:value_objects:dimensions" href="#usage:value_objects:dimensions">Dimensions</a>

The `Dimensions` class represents the dimensions of an object in pixels. It is a read-only value object with two properties: `width` and `height`.

```php
$dimensions = new Dimensions(100, 150);

$width = $dimensions->width; // 100
$height = $dimensions->height; // 150
```

##### <a id="usage:value_objects:point" href="#usage:value_objects:point">Point</a>

The `Point` class represents a 2D coordinate. It is a read-only value object with two properties: `x` and `y`.

```php
$point = new Point(100, 150);

$x = $point->x; // 100
$y = $point->y; // 150
```

##### <a id="usage:value_objects:points" href="#usage:value_objects:points">Points</a>

The `Points` class represents an ordered collection of points. It is used by drawing operations to define shapes, lines, and paths. The class implements `Countable` and `IteratorAggregate`, allowing it to be counted and iterated over directly.

```php
$square = new Points(
    new Point(0, 0),     // top-left
    new Point(100, 0),   // top-right
    new Point(100, 100), // bottom-right
    new Point(0, 100),   // bottom-left
);

foreach ($square as $point) {
    // ...
}
```

The following additional methods are available:

| Method                            | Description                                                                                                          |
|-----------------------------------|----------------------------------------------------------------------------------------------------------------------|
| getPoints()                       | Returns the points contained in the collection                                                                       |
| getDimensions()                   | Returns a `Dimensions` instance representing the bounding box containing the points                                  |
| fitTo($dimensions)                | Returns a new set of points fitted to the given dimensions while preserving the aspect ratio and normalized to `0,0` |

