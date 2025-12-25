# Image manipulation

--------------------------------------------------------

* [Usage](#usage)
    - [Image information](#usage:image_information)
    - [Image manipulation](#usage:image_manipulation)
    - [Image metadata](#usage:image_metadata)
        - [XMP](#usage:image_metadata:xmp)
            - [Reading](#usage:image_metadata:xmp:reading)

--------------------------------------------------------

The pixel library allows you to manipulate images through a consistent API using either GD or ImageMagick.

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

First you'll have to decide whether to use GD or ImageMagick. In this example we'll use the ImageMagick processor.

```
$image = new Gd('image.png');
```

#### <a id="usage:image_information" href="#usage:image_information">Image information</a>

The `getHeight` method returns the height of your image.

```
$height = $image->getHeight();
```

The `getWidth` method returns the width of your image.

```
$width = $image->getWidth();
```

The `getDimensions` method returns an array containing both the height and width of your image.

```
$dimensions = $image->getDimensions();
```

The `getTopColors` method returns an array of the top `n` colors in an image, represented as `Color` class instances. By default, it returns the top five colors, but you can specify a custom number of colors to return.

```
$topColors = $image->getTopColors();

foreach ($topColors as $color) {
    var_dump($color->toHexString());
}
```

The following methods are available on the `Color` class:

| Method                             | Description                                                                           |
|------------------------------------|---------------------------------------------------------------------------------------|
| getRed()                           | Returns the red value (0-255)                                                         |
| getGreen()                         | Returns the green value (0-255)                                                       |
| getBlue()                          | Returns the blue value (0-255)                                                        |
| getAlpha()                         | Returns the alpha value (0-255)                                                       |
| toHexString($withAlpha = false)    | Returns a hex string representation of the color (e.g. "#FF0000")                   |
| toRgbString()                      | Returns a rgb string representation of the color (e.g. "rgb(255, 0, 0)")            |
| toRgbaString()                     | Returns a rgb string representation of the color (e.g. "rgba(255, 0, 0, 0.5)")      |
| toHslString()                      | Returns a hsl string representation of the color (e.g. "hsl(0, 100.0%, 50.0%)")       |
| toHslaString()                     | Returns a hsla string representation of the color (e.g. "hsl(0, 100.0%, 50.0%, 0.5)") |
| toHwbString()                      | Returns a hwb string representation of the color (e.g. "hwb(0 0.0% 0.0%)")            |
| toHwbaString()                     | Returns a hwba string representation of the color (e.g. "hwb(0 0.0% 0.0% / 0.5)")     |

> Note that if you want the best color accuracy then you should use the `ImageMagick` class.

#### <a id="usage:image_manipulation" href="#usage:image_manipulation">Image manipulation</a>

The `snapshot` method allows you to create a snapshot of your image.

```
$image->snapshot();
```

The `restore` method allows you to restore an image snapshot.

```
$image->restore();
```

The `apply` method allows you to apply an image operation to your image.

```
$image->apply(new Sharpen);
$image->apply(new Border(new Color(0, 0, 0, 127), size: 10));
```

Here are all of the included image operations:

| Class                | Description                                                    | Gd | ImageMagick |
|----------------------|----------------------------------------------------------------|----|-------------|
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
| Resize               | Resizes the image to the chosen size                           | ✓  | ✓           |
| Rotate               | Rotates the image                                              | ✓  | ✓           |
| Saturation           | Adjusts the saturation of the image (-100 to 100)              | ✓  | ✓           |
| Sepia                | Applies a sepia filter to the image                            | ✓  | ✓           |
| Sharpen              | Sharpens the image                                             | ✓  | ✓           |
| Temperature          | Adjusts the color temperature of the image (-100 to 100)       | ✓  | ✓           |
| Temperature          | Applies a custom watermark to the image                        | ✓  | ✓           |

You can also create your own custom image operations by implementing the `OperationInterface`.

```
class MyOperation implements OperationInterface
{
    #[Override]
	public function apply(object &$imageResource): void
	{
        // Do your custom image operations here
    }
}
```

The `getImageBlob` method returns the raw binary image data.

```
$image = $image->getImageBlob();

// You can also tell it to return a different image type

$image = $image->getImageBlob('jpg');

// You can also adjust the image quality in percent (default is 95%)

$image = $image->getImageBlob('jpg', 70);
```

As the name suggests the `save` method will save your edited image to disk.

```
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

> Note that the XMP reader relies on [`FFI`](https://www.php.net/manual/en/book.ffi.php) and `libexempi`. The reader will attempt to auto-detect the shared library, but depending on your setup, you may need to specify it manually.

```
$reader = new XmpReader('image.tif');

// You can specify the shared library if needed

$reader = new XmpReader('image.tif', 'libexempi.so.8');
```

The `getXml` method returns all the XMP data as XML.

```
$xml = $reader->getXml();

// You can also cast the reader object to a string

$xml = (string) $reader;
```

The `getProperties` method returns all the XMP properties as objects.

```
$properties = $reader->getProperties();

// You can also select the properties from a specific namespace

$properties = $reader->getProperties('http://purl.org/dc/elements/1.1/');
```

The `getProperty` method allows you to get a specific property. The first parameter is the property namespace and the second is the property name.

```
$property = $reader->getProperty('http://purl.org/dc/elements/1.1/', 'title');

echo $property->value;
```
