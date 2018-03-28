# Upgrading

--------------------------------------------------------

* [Framework](#framework)
	- [Response](#framework:response)
	- [Validation](#framework:validation)

--------------------------------------------------------

This guide takes you through the steps needed to migrate from Mako `5.4.x` to `5.5.x`.

--------------------------------------------------------

<a id="framework"></a>

### Framework

<a id="framework:response"></a>

#### Response

All methods that were deprecated in 5.4 have been removed. Check out the [response](:base_url:/docs/:version:/routing-and-controllers:response) docs for information on how to upgrade your application.

<a id="framework:validation"></a>

#### Validation

The validation library has been rewritten from scratch to provide an easier way to validate arrays and a simpler way of adding advanced custom validation rules.

There are no breaking changes if you're just using the included validation rules. If you have implemented custom rules then you'll have to re-implement them using the new `RuleInterface`. Check out the [documentation](:base_url:/docs/:version:/learn-more:validation#custom_rules) for details.
