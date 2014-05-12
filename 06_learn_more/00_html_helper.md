# HTML helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The HTML helper can be used to generate nearly any HTML tag.

--------------------------------------------------------

<a id="usage"></a>

### Usage

First we'll need to create a HTML instance.

	$html = new HTML;

	// You can make it return XHTML by setting the $xhtml parameter to TRUE

	$html = new HTML(true);

The ```tag``` method can be used to generate nearly any HTML tag.

	// Will print "<br>"

	echo $html->tag('br');

	// Will print <span>Hello, world!</span>

	echo HTML::tag('span', [], 'Hello, world!');

	// Will print <span id="foo" class="bar">Hello, world!</span>

	echo $html->tag('span', ['id' => 'foo', 'class' => 'bar'], 'Hello, world!');

The ```audio``` method generates HTML5 tags for audio files.

	echo $html->audio('http://example.org/file.mp3');

	echo $html->audio(['http://example.org/file.mp3', 'http://example.org/file.ogg']);

The ```video``` method generates HTML5 tags for video files.

	echo $html->video('http://example.org/file.mp4');

	echo $html->video(['http://example.org/file.mp4', 'http://example.org/file.ogg']);

The ```ul``` method generates HTML tags for an unordered list.

	$items = ['apple', 'banana', 'orange'];

	echo $html->ul($items);

	echo $html->ul($items, ['class' => 'my_list']);

	// It also supports multi-dimensional arrays

	$items = ['foo', 'bar', ['bar', 'foo']];

	echo $html->ul($items);

The ```ol``` method generates HTML tags for a ordered list.

	$items = ['apple', 'banana', 'orange'];

	echo $html->ol($items);

	echo $html->ol($items, ['class' => 'my_list']);

	// It also supports multi-dimensional arrays

	$items = ['foo', 'bar', ['bar', 'foo']);

	echo $html->ol($items);

The ```registerTag``` method allows you to register custom tag methods.

	HTML::registerTag('p', function($html, $content, array $attributes = [])
	{
		return $html->tag('p', $attributes, $content);
	});

	// Will print <p>Hello</p>

	$html->p('Hello!');

	// Will print <p class="foo">Hello</p>

	$html->p('Hello!', ['class' => 'foo']);