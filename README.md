Jasny Session middleware
===

[![Build Status](https://travis-ci.org/jasny/session-middleware.svg?branch=master)](https://travis-ci.org/jasny/session-middleware)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/jasny/session-middleware/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/jasny/session-middleware/?branch=master)
[![Code Coverage](https://scrutinizer-ci.com/g/jasny/session-middleware/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/jasny/session-middleware/?branch=master)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/a1a1745c-1272-46a3-9567-7bbb52acda5a/mini.png)](https://insight.sensiolabs.com/projects/a1a1745c-1272-46a3-9567-7bbb52acda5a)
[![BCH compliance](https://bettercodehub.com/edge/badge/jasny/session-middleware?branch=master)](https://bettercodehub.com/)
[![Packagist Stable Version](https://img.shields.io/packagist/v/jasny/session-middleware.svg)](https://packagist.org/packages/jasny/session-middleware)
[![Packagist License](https://img.shields.io/packagist/l/jasny/session-middleware.svg)](https://packagist.org/packages/jasny/session-middleware)

Using superglobals like `$_SESSION` object makes it difficult to test an application as global variables can have
unexpected side effects. Using superglobals undermines the effort of using dependecy injection and using containers.

The middleware creates uses PHP [SessionHandlerInterface](http://php.net/manual/en/class.sessionhandlerinterface.php)
objects directly. By default it uses the `ext/session` handler as well as the global session name and cookie
parameters.

The middleware complies with [PSR-15](https://www.php-fig.org/psr/psr-15/). It will also work as double pass middleware.

Installation
---

    composer require jasny/session-middleware

Usage
---

```php
use Jasny\SessionMiddleware;

$router->add(new SessionMiddleware());
$response = $router->handle($request);
```

Get the session object from the PSR-7 ServerRequest object and use it as array

```php
$session = $request->getAttribute('session');
$session['foo.bar'] = 10;

if (isset($session['foo.user'])) {
  // ...
}
```

Use `$session->abort()` to abort writing the changes and `$session->destroy()` to destroy the session.

### Flash

The session flash object can be used to pass a message to the next request. It is automatically removed from the session
after it is used. A typical use case is to store information in a database, than redirect to a page and showing a
success message. Or if the information could not be saved, to show an error message.

The flash information contains a type (e.g. `success`, `error`, `info`, `warning`) and a message. Optionally a
content type can be specified for the message. This defaults to `text/plain`.

```php
use Jasny\Session;

$flash = new Session\Flash($session);
$session->set('success', 'The information has been saved');
```

In the next request

```php
use Jasny\Session;

$flash = new Session\Flash($session);
echo $flash->getType(), ': ', $flash->getMessage();
```

_Displaying the message is outside the scope of this library._

### Dependency injection

If you want to use modified services, you can use the `with...` methods for dependency injection.

```php
use Jasny\Session;
use Jasny\SessionMiddleware;

$sessionFactory = new Session(); // Session implementation has factory method

$middleware = (new SessionMiddleware())
  ->withSessionParams($sessionName, $cookieParams)
  ->withSessionHandler($handler)
  ->withSessionFactory($sessionFactory)
  ->withEncoder($encode, $decode);
```

_Note that `SessionMiddleware` is an immutable object. The `with...` methods clone the middleware and only modify the
clone._

If the session handler doesn't implement `SessionIdInterface`, you need to pass a service that can generate session ids
as second parameter.

A `Session` object has a factory method `create(string id, array data)` to create a new session. This method can be used
to create a base `Session` that is copied on use. This prevents having to have to write and extend a separate factory
class.

You should always depend on `SessionInterface` and `SessionFactoryInterface` rather than the `Session` class.

The `Session\Flash` also has a factory method. You can use a `Session\Flash` object for dependency injection. You should
always depend on `Session\FlashInterface` and `Session\FlashFactoryInterface` rather than the `Session\Flash` class.


Testing
---

When running tests, you can injecting a new `Session` object in the request before passing it to the middleware.

```php
$session = new Jasny\Session([
  'foo.user' => 'john@example.com'
]);

$response = $router->handler($request->withAttribute('session', $session));
```

The session middleware will ignore a session object that wasn't created by the middleware.
