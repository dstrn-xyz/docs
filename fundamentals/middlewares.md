# middleware

- [introduction](#introduction)
- [defining middleware](#defining-middleware)
- [the next function](#the-next-function)
- [halting the chain](#halting-the-chain)
- [registering middleware](#registering-middleware)
- [middleware parameters](#middleware-parameters)
- [hot reloading](#hot-reloading)

<a name="introduction"></a>

## introduction

middleware provides a convenient mechanism for inspecting and filtering http requests entering your application. for example an authentication middleware might verify the user is logged in. if they are not the middleware redirects them to the login screen. if they are authenticated the middleware allows the request to proceed deeper into the application.

<a name="defining-middleware"></a>

## defining middleware

all custom middleware lives in the `middlewares` directory. you define middleware as a class with async methods. each method acts as an independent middleware handler.

```javascript
// middlewares/AuthMiddleware.js
export default class AuthMiddleware {
  async requireAuth(req, next) {
    if (!Auth.check()) {
      return redirect('/login', true);
    }
    return next();
  }

  async requireGuest(req, next) {
    if (Auth.check()) {
      return redirect('/home', true);
    }
    return next();
  }
}
```

you can generate a middleware stub using the cli.

```bash
dstrn make:middleware AuthMiddleware
```

<a name="the-next-function"></a>

## the next function

every middleware method receives the `req` object and a `next` closure. you must call `next()` to pass control to the next middleware in the execution chain. if you do not call `next()` the request halts at that middleware and the framework assumes you have handled the response.

some middleware may need access to the response object. you can declare `res` as the second argument and `next` as the third.

```javascript
export default class CustomMiddleware {
  async intercept(req, res, next) {
    res.set('X-Custom-Header', 'dstrn');
    return next();
  }
}
```

<a name="halting-the-chain"></a>

## halting the chain

to halt the request and prevent it from reaching the controller simply return a response helper like `redirect()`, `abort()`, or `json()` instead of calling `next()`.

```javascript
// middlewares/AdminMiddleware.js
export default class AdminMiddleware {
  async requireAdmin(req, next) {
    if (!Auth.check()) return redirect('/login');
    if (Auth.user().power < 5) return abort(403, 'unauthorized access');
    
    return next();
  }
}
```

the middleware runner strictly monitors the response state. if a middleware sends headers or ends the writable stream the chain is immediately broken even if `next()` was accidentally called.

<a name="registering-middleware"></a>

## registering middleware

middleware is not registered globally. you attach it directly to your routes using the string syntax `ClassName@method`. the router resolves these strings automatically.

you can apply middleware to a single route.

```javascript
Route.middleware('AuthMiddleware@requireAuth').get('/profile', 'app.UserProfileController@show');
```

you can apply middleware to a group of routes.

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.get('/dashboard', 'app.IndexController@dashboard');
  auth.get('/library', 'app.IndexController@library');
});
```

you can chain multiple middleware by providing an array.

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth', 'AdminMiddleware@requireAdmin'] }, (admin) => {
  admin.get('/users', 'admin.AdminController@list');
});
```

you can also define routes using an inline array where all items except the last are treated as middleware.

```javascript
Route.post('/device/approve', ['AuthMiddleware@requireAuth', 'auth.AuthController@approvePost']);
```

<a name="middleware-parameters"></a>

## middleware parameters

dframework does not support passing parameters directly through the middleware string syntax. if your middleware requires dynamic configuration you should define separate methods on the class or pass data via the `req` object.

```javascript
export default class DeviceMiddleware {
  async verifyDevice(req, next) {
    const devices = await Device.where({ user_id: Auth.user().id, revoked: false }).get();
    
    // attach data to the request for the controller
    req.devices = devices;
    
    return next();
  }
}
```

<a name="hot-reloading"></a>

## hot reloading

in the `local` environment the framework watches the `middlewares` directory. when you modify a middleware file the runner automatically invalidates its cache and reloads the class. this means your middleware logic hot reloads instantly without requiring a server restart.

<a name="rate-limiting"></a>

## rate limiting

the built in `RateLimiter` uses a sliding window algorithm. it tracks request counts per key within a configurable time window, automatically cleans up expired entries, and sets standard rate limit response headers.

```javascript
import { RateLimiter } from 'dframework';

const limiter = new RateLimiter({
  windowMs: 60 * 1000, // 1 minute
  max: 60,             // 60 requests per window
  message: 'too many requests, please slow down',
});

Route.middleware(limiter.middleware()).group({ prefix: '/api' }, (Route) => {
  Route.post('/messages', 'MessageController@store');
  Route.post('/uploads',  'UploadController@store');
});
```

when the limit is exceeded, the response includes `Retry-After`, `X-RateLimit-Limit`, and `X-RateLimit-Remaining` headers alongside a `429` status.

| option           | default               | description                                        |
| ---------------- | --------------------- | -------------------------------------------------- |
| `windowMs`       | `60000`               | time window in milliseconds                        |
| `max`            | `60`                  | maximum requests per window                        |
| `message`        | `'too many requests'` | message returned on limit exceeded                 |
| `keyGenerator`   | client ip             | `(req) => string`, custom grouping key             |
| `trustedProxies` | `[]`                  | trusted proxy ips for `x-forwarded-for` resolution |

a custom `keyGenerator` is useful for per user limiting:

```javascript
const limiter = new RateLimiter({
  windowMs: 60 * 1000,
  max: 20,
  keyGenerator: (req) => Auth.user()?.id || req._req.socket.remoteAddress,
});
```
