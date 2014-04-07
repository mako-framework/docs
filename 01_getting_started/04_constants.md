# Constants

--------------------------------------------------------

* [Defaults](#defaults)
* [Modifiers](#modifiers)

--------------------------------------------------------

<a id="defaults"></a>

### Defaults

The following constants are defined by the Mako framework:

| Name                         | Description                                                                             |
|------------------------------|-----------------------------------------------------------------------------------------|
| MAKO_VERSION                 | Holds the Mako framework version.                                                       |
| MAKO_APPLICATION_PATH        | Defines the path to the app directory (without trailing slash).                         |
| MAKO_START                   | Starting point of the framework execution in ms.                                        |
| MAKO_IS_WINDOWS              | Is set to TRUE if Mako runs on a Windows system and FALSE if not.                       |

--------------------------------------------------------

<a id="modifiers"></a>

### Modifiers

The following constants can modify the behavior of the framework if defined:

| Name                         | Description                                                                             |
|------------------------------|-----------------------------------------------------------------------------------------|
| MAKO_XHTML                   | If this constant is defined then the HTML helper will return XHTML compatible tags.     |