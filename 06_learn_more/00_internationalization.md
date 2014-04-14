# Internationalization

--------------------------------------------------------

* [Language files](#language_files)
* [Usage](#usage)
* [Language routing](#language_routing)

--------------------------------------------------------

The I18n class makes it easy to create a multilingual web application.

The name of a language pack should use the following convention: ```en_US``` ([ISO 639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) language code and [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code).

The default language of your application is defined in the ```app/config/application.php``` file.

> All language packs must be installed in the ```app/i18n``` directory.

--------------------------------------------------------

<a id="language_files"></a>

### Language files

Mako language files are just simple arrays:

	<?php

	// examples.php

	return 
	[
		'have_10_apples' => 'I have 10 apples',
		'have_n_apples'  => 'I have %d apples',
		'my_name_is'     => 'Hi! My name is %s',
	];

--------------------------------------------------------

<a id="usage"></a>

### Usage

Setting the language you want to use use is done by calling the ```setLanguage``` method.

	$this->i18n->setLanguage('no_NO');

The ```getLanguage``` returns the name of the current language pack.

	$language = $this->i18n->getLanguage();

Fetching a string is done with the ```get``` method. If no string is found then the key is returned.

	// Will print "I have 10 apples"

	echo $this->i18n->get('examples.have_10_apples');

	// Will print "I have 20 apples"

	echo $this->i18n->get('examples.have_n_apples', [20]);

You can check if a string exists using the ```has``` method.

	$exists = $this->i18n->has('my_key');

The ```pluralize``` method returns the plural form of the chosen noun. Unlike most other frameworks Mako will use language based inflection rules when pluralizing words.

> The pluralize method will only work if the chosen language pack includes inflection rules.

	echo $this->i18n->pluralize('woman'); // Will print "women"

	echo $this->i18n->pluralize('woman', 1); // Will print "woman"

	echo $this->i18n->pluralize('woman', 2); // Will print "women"

You can also pluralize words in translated strings.
	 
	return array
	(
		'new_messages' => 'You have %1$u new <pluralize:%1$u>message</pluralize>.',
	);

We can now enjoy simple pluralization in our views
	 
	{{$i18n->get('ui.new_messages', [1])}} // Will print "You have 1 new message."

	{{$i18n->get('ui.new_messages', [10])}} // Will print "You have 10 new messages."

--------------------------------------------------------

<a id="language_routing"></a>

### Language routing

Language route prefixes can be configured in the ```app/config/application.php``` config file.

	'languages' => array
	(
		'no' => 'nb_NO',
	),

Visiting ```http://example.org/no/foo/bar``` set the language to norwegian before executing the ```/foo/bar``` route.