# HTML helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The HTML helper can be used to generate nearly any HTML tag.

> The HTML helper will return HTML5 tags but you can make it return XHTML compatible tags by defining the ```MAKO_XHTML``` constant.

--------------------------------------------------------

<a id="usage"></a>

### Usage

The ```tag``` method can be used to generate nearly any HTML tag.

	// Will print "<br>"

	echo HTML::tag('br');

	// Will print <span>Hello, world!</span>

	echo HTML::tag('span', [], 'Hello, world!');

	// Will print <span id="foo" class="bar">Hello, world!</span>

	echo HTML::tag('span', ['id' => 'foo', 'class' => 'bar'], 'Hello, world!');

The ```audio``` method generates HTML5 tags for audio files.

	echo HTML::audio('http://example.org/file.mp3');

	echo HTML::audio(['http://example.org/file.mp3', 'http://example.org/file.ogg']);

The ```video``` method generates HTML5 tags for video files.

	echo HTML::video('http://example.org/file.mp4');

	echo HTML::video(['http://example.org/file.mp4', 'http://example.org/file.ogg']);

The ```ul``` method generates HTML tags for an unordered list.

	$items = ['apple', 'banana', 'orange'];

	echo HTML::ul($items);

	echo HTML::ul($items, ['class' => 'my_list']);

	// It also supports multi-dimensional arrays

	$items = ['foo', 'bar', ['bar', 'foo']];

	echo HTML::ul($items);

The ```ol``` method generates HTML tags for a ordered list.

	$items = ['apple', 'banana', 'orange'];

	echo HTML::ol($items);

	echo HTML::ol($items, ['class' => 'my_list']);

	// It also supports multi-dimensional arrays

	$items = ['foo', 'bar', ['bar', 'foo']);

	echo HTML::ol($items);

The ```registerTag``` method allows you to register custom tag methods.

	HTML::registerTag('p', function($content, array $attributes = [])
	{
		return HTML::tag('p', $attributes, $content);
	});

	// Will print <p>Hello</p>

	HTML::p('Hello!');

	// Will print <p class="foo">Hello</p>

	HTML::p('Hello!', ['class' => 'foo']);