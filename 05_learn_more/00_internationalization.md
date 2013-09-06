# Internationalization

--------------------------------------------------------

* [Language files](#language_files)
* [Usage](#usage)

--------------------------------------------------------

The I18n class makes it easy to create a multilingual web application. You can add as many languages as you want.

The language packs are located in the ```app/resources/i18n``` directory and the files for each language are located in a subdirectory.

The name of a language pack should use the following convention: ```en_US``` ([ISO 639-1](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes) language code and [ISO 3166-1 alpha-2](http://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) country code).

The default language of your application is defined in the ```app/config/application.php``` file.

--------------------------------------------------------

<a id="language_files"></a>

### Language files

Mako language files are just simple arrays:

	<?php

	// examples.php

	return array
	(
		'have_10_apples' => 'I have 10 apples',
		'have_n_apples'  => 'I have %d apples',
		'my_name_is'     => 'Hi! My name is %s',
	);

--------------------------------------------------------

<a id="usage"></a>

### Usage

Setting the language you want to use use is done by calling the ```language``` method. The method also returns the name of the current language.

	I18n::language('no_NO'); // Setting the language to Norwegian

	$language = I18n::language(); // The value of $language is now "no_NO"

Fetching a string is done with the ```get``` method. If no string is found then the key is returned. You can also use the ```__``` convenience function.

	// Will print "I have 10 apples"

	echo I18n::get('examples.have_10_apples');

	// Will print "I have 20 apples"

	echo I18n::get('examples.have_n_apples', array(20));

	// Will print "Hi! My name is Iroquois Pliskin"

	echo __('examples.my_name_is', array('Iroquois Pliskin'));

You can check if a string exists using the ```has``` method.

	$exists = I18n::has('my_key');

The ```pluralize``` method returns the plural form of the chosen noun. Unlike most other frameworks Mako will use language based inflection rules when pluralizing words.

> The pluralize method will only work if the chosen language pack includes inflection rules.

	echo I18n::pluralize('woman'); // Will print "women"

	echo I18n::pluralize('woman', 1); // Will print "woman"

	echo I18n::pluralize('woman', 2); // Will print "women"

You can also pluralize words in translated strings:

	// In a language file caleld ui.php
	 
	return array
	(
		'new_messages' => 'You have %1$u new <pluralize:%1$u>message</pluralize>.',
	);
	 
	// In a view (using the template syntax)
	 
	{{__('ui.new_messages', array(1)}} // Will print "You have 1 new message."

	{{__('ui.new_messages', array(10)}} // Will print "You have 10 new messages."