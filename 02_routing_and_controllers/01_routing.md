# Routing

--------------------------------------------------------

* [Basics](#basics)
* [Advanced routing](#advanced_routing)
* [Reverse routing](#reverse_routing)

--------------------------------------------------------

The Mako router allows you to map URLs to controller and/or closure actions. It also allows you to perform reverse routing so that you don't have to hardcode strings in your views. 

--------------------------------------------------------

<a id="basics"></a>

### Basics

Routes are registered in the ```app/routes.php``` file. There are two variables avaiable in the scope, ```$routes``` (the route collection) and ```$app``` (the application instance).

	$routes->get('/', 'app\controllers\Home::welcome');