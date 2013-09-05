# Cookies

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The cookie class provides a simple and consistent way to set and read cookies. All cookies set using the Cookie class will be signed.

The benefit of using signed cookies over regular cookies is that their values can not be tampered with on the client site.

--------------------------------------------------------

<a id="usage"></a>

### Usage

#### Setting cookies

Setting a signed cookie is done by using the ```set``` method. The available options are ```path```, ```domain```, ```secure``` and ```httponly```.

	// Sets signed a cookie that will expire when the browser closes.

	Cookie::set('foobar', 'my value');

	// Sets signed a cookie that will expire after 10 hours

	Cookie::set('foobar', 'my value', (3600 * 10));

	// Sets a signed "httponly" cookie that will expire after 10 hours

	Cookie::set('foobar', 'my value', (3600 * 10), array('httponly' => true));

#### Reading cookies

Reading a signed cookie is done using the ```get``` method. Unsigned cookies or cookies with an invalid signature will be ignored.

	// Reads the 'foobar' cookie. If the cookie doesn't exist then $value will be set to NULL

	$value = Cookie::get('foobar');

	// Reads the 'foobar' cookie. If the cookie doesn't exist then $value will be set to '1234'

	$value = Cookie::get('foobar', '1234');

#### Deleting cookies

Deleting cookies is done by using the ```delete``` method.

	Cookie::delete('foobar');

	// You can also set options when deleting cookies

	Cookie::delete('foobar', array('httponly' => true));