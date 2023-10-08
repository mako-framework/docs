# Retry helper

--------------------------------------------------------

* [Usage](#usage)

--------------------------------------------------------

The retry helper simplifies automatic retries of actions that might occasionally fail.

--------------------------------------------------------

By default the retry class will attempt to execute the closure up to `5` times and wait `50000` microseconds between each attempt. If the action fails too many times then the exception will be thrown.

```
$returnValue = (new Retry(function () {
    // Some action that might fail
}))();
```

You can override the default values using the `$attempts` and `$wait` constructor parameters or by using the `setAttempts` and `setWait` methods. You can also enable exponential wait times by setting the `$exponentialWait` constructor parameter to `true` or by using the `exponentialWait` method.

```
$returnValue = (new Retry(function () {
    // Some action that might fail
}))
->setAttempts(4)
->setWait(25000)
->exponentialWait()();
```

It is also possible to short-circuit the retry loop using a "decider" callable. If the callable returns `true` then execution will continue and if it returns `false` the exception will be thrown right away.

```
$returnValue = (new Retry(function () {
    // Some action that might fail
}))
->setDecider(fn ($e) => ($e instanceof SpecialException::class) === false)();
```

You can also set the decider using the `$decider` constructor parameter.