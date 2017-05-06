# Upgrading

--------------------------------------------------------

* [Framework](#framework)
	- [Cache](#framework:cache)
	- [Gatekeeper](#framework:gatekeeper)
	- [Response](#framework:response)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako ```5.1.x``` to ```5.2.x```.

--------------------------------------------------------

<a id="framework"></a>

### Framework

<a id="framework:cache"></a>

#### Cache

The `CacheManager::instance` method will now return an instance of `mako\cache\stores\StoreInterface` instead of a `mako\cache\Cache` instance.

<a id="framework:gatekeeper"></a>

#### Gatekeeper

Gatekeeper has been rewritten from scratch to make it easier to extend and to make it possible add authentication adapters.

The library has been moved from the `mako\auth` namespace to the `mako\gatekeeper` namespace and some classes have been renamed and/or moved internally so some changes will have to be made if your application is using type hinted dependency injection.

If you're using Gatekeeper through the container aware trait then no changes will have to made except for updating the namespace of the `User` and `Group` models.

There has also been a couple of method name changes:

* `Gatekeeper::getUserProvider()` is now named `Gatekeeper::getGroupRepository()`.
* `Gatekeeper::getGroupProvider()` is now named `Gatekeeper::getGroupRepository()`.

The `Gatekeeper::basicAuth()` method will now always return a boolean. This allows you to set your own message (The `WWW-Authenticate` header and `401` status code will still be set for you automatically).

	if($this->gatekeeper->basicAuth() === false)
	{
		return 'Authentication required.';
	}

<a id="framework:response"></a>

#### Response

Response headers will now be set with the case that they where defined with. If your clients adhere to the [spec](https://www.w3.org/Protocols/rfc2616/rfc2616-sec4.html#sec4.2) then no changes will have to be made.
