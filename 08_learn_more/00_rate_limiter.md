# Rate limiter

--------------------------------------------------------

* [Usage](#usage)
    - [Middleware](#usage:middleware)
    - [Advanced](#usage:advanced)
    - [Picking the right store](#usage:picking-the-right-store)

--------------------------------------------------------

The Mako rate limiter library provides a flexible and efficient way to control the number of times a specific action can be performed within a given time window. This helps prevent abuse, enforce usage quotas, and ensure fair resource distribution in your application.

By default, the library includes an HTTP user context, allowing you to rate-limit actions based on individual users, or IP addresses.

For storing rate-limiting data, Mako supports multiple backends:

* [APCu](https://www.php.net/manual/en/book.apcu.php): The default store, ideal for applications running on a single-instance setup, as it provides fast, in-memory storage without requiring an external service.
* [Redis](https://redis.io): Recommended for high-traffic environments with multiple application replicas, as it enables centralized rate-limiting enforcement across all instances, ensuring consistency.

With its simple API and configurable storage options, the Mako rate limiter makes it easy to implement rate limits tailored to your application’s needs.

--------------------------------------------------------

### <a id="usage" href="#usage">Usage</a>

The Mako rate limiter library offers two ways to enforce rate limits, depending on your needs:

1) The library includes an HTTP [middleware](:base_url:/docs/:version:/routing-and-controllers:routing#route_middleware) that makes it easy to apply rate limits to incoming requests with minimal configuration. This is the preferred approach for most use cases, as it seamlessly integrates into your application’s request lifecycle. Simply register the middleware, configure the rate limits, and let it handle the enforcement automatically.

2) If your application requires more granular control over rate limiting—such as applying limits to non-HTTP actions (e.g., CLI commands, API tokens, or job queues)—you can interact with the library directly. This allows you to manually define rate limits, check usage, and increment counters programmatically.

#### <a id="usage:middleware" href="#usage:middleware">Middleware</a>

The following example demonstrates how to apply rate limiting to an API endpoint, restricting access based on either an authenticated user or a single IP address. The limit is set to 5 requests within a 24-hour window.

If a user or IP exceeds this limit, the application will throw a `TooManyRequestsException`. This results in a `429 Too Many Requests` response, along with a `Retry-After` header that informs the client when they can attempt the request again.

The `resetAfter` parameter accepts a `DateInterval` instance or a valid [date time string](https://www.php.net/manual/en/datetime.formats.php#datetime.formats.relative).

```
<?php

namespace app\http\controllers;

use mako\http\response\builders\JSON;
use mako\http\routing\attributes\Middleware;
use mako\throttle\http\routing\middleware\RateLimiter;

#[Middleware(RateLimiter::class, maxRequests: 5, resetAfter: '24 hours')]
class Endpoint
{
    public function __invoke(): JSON
    {
        return new JSON(['greeting' => 'Hello, world!']);
    }
}
```

#### <a id="usage:advanced" href="#usage:advanced">Advanced</a>

To integrate rate limiting into your application, you can inject a rate limiter instance into your dependencies using the `RateLimiterInterface`.

```
<?php

use mako\throttle\RateLimiterInterface;

class Dependent
{
    public function __construct(
        protected RateLimiterInterface $rateLimiter
    ) {
    }
}
```

The `isLimitReached` method allows you to check whether the maximum number of allowed attempts for a specific action has been reached. The first parameter is a string that uniquely identifies the action you want to limit. The second parameter is an integer that defines the maximum number of times the action can be performed before being restricted.

The method will return `true` if the limit has been reached and `false` if it hasn't.

```
$limitReached = $rateLimiter->isLimitReached('send_email', 10);
```

The `getRemaining` method returns the number of remaining attempts for the action before the limit has been reached. The first parameter is a string that uniquely identifies the action you want to limit. The second parameter is an integer that defines the maximum number of times the action can be performed before being restricted.

```
$remainingAttempts = $rateLimiter->remainingAttempts('send_email', 10);
```

The `getRetryAfter` method provides a convenient way to determine when a rate limit for a specific action will expire. When called, it returns a `DateTimeInterface` instance that represents the exact time and date when the limit will reset, allowing the action to be attempted again.

If there isn’t an active limiter for the specified action, the method will return `null`.

```
$retryAfter = $rateLimiter->getRetryAfter('send_email');
```

The `increment` method increments the counter for the specified action. The first parameter is a string that uniquely identifies the action you want to limit. The second parameter is a `DateInterval` instance.

After incrementing the counter, the method returns the updated count, making it possible to immediately check how many attempts have been made. This is particularly useful when implementing logic that needs to respond differently based on how close the user is to reaching the limit.

```
$attempts = $rateLimiter->increment('send_email', DateInterval::createFromDateString('10 minutes'));
```

#### <a id="usage:picking-the-right-store" href="#usage:middlpicking-the-right-storeeware">Picking the right store</a>

By default, the library uses the `APCu` store, which provides fast, in-memory storage without requiring an external service. This makes it an ideal choice if you're running a single-instance application.

However, if your application is deployed across multiple load-balanced replicas, you'll need a centralized store to track rate limits consistently across instances. In this case, the provided `Redis` store is the recommended option, as it ensures that limits are enforced application-wide.

Additionally, if you need a custom storage solution, you can implement your own store by creating a class that implements the `StoreInterface`. This allows you to integrate with other storage backends tailored to your specific requirements.

To use the `Redis` store or a custom implementation, you'll need to override the `getStore` method of the `RateLimiterService` class.

```
<?php

namespace app\services;

use Closure;
use mako\application\services\RateLimiterService as BaseRateLimiterService;
use mako\throttle\stores\Redis;
use Override;

/**
 * {@inheritDoc}
 */
class RateLimiterService extends BaseRateLimiterService
{
	/**
	 * {@inheritDoc}
	 */
    #[Override]
	protected function getStore(): Closure|string
	{
		return Redis::class;
	}
}
```
