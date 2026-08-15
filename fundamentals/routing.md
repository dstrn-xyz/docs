# routing

- [routing](#routing)
  - [introduction](#introduction)
  - [basic routes](#basic-routes)
    - [available methods](#available-methods)
  - [route parameters](#route-parameters)
  - [named routes](#named-routes)
  - [route groups](#route-groups)
    - [prefix](#prefix)
    - [middleware](#middleware)
    - [combined attributes](#combined-attributes)
    - [middleware ordering](#middleware-ordering)
  - [domain and port routing](#domain-and-port-routing)
    - [domain routing](#domain-routing)
    - [port routing](#port-routing)
    - [chainable builders](#chainable-builders)
    - [parameterized domains](#parameterized-domains)
    - [overlapping route names across domains](#overlapping-route-names-across-domains)
  - [inline middleware chains](#inline-middleware-chains)
  - [route modifiers](#route-modifiers)
    - [csrf](#csrf)
    - [shield](#shield)
    - [log](#log)
    - [views](#views)
    - [profile](#profile)
  - [controller string syntax](#controller-string-syntax)
  - [multiple route files](#multiple-route-files)

<a name="introduction"></a>

## introduction

routes are defined in the `routes` directory. dframework automatically discovers and loads every `.js` file in that directory at boot time without any manual imports. inside each route file you have access to the global `Route` facade which proxies directly to the internal router instance.

<a name="basic-routes"></a>

## basic routes

the simplest route accepts a url path and a controller string pointing to the method that should handle it.

```javascript
// routes/web.js
import { Route } from 'dframework';

Route.get('/dashboard', 'app.IndexController@dashboard');
Route.post('/register', 'auth.AuthController@register');
Route.put('/profile', 'app.UserProfileController@update');
Route.delete('/session', 'auth.AuthController@logout');
```

<a name="available-methods"></a>

### available methods

the router supports four http verbs.

```javascript
Route.get(path, handler);
Route.post(path, handler);
Route.put(path, handler);
Route.delete(path, handler);
```

<a name="route-parameters"></a>

## route parameters

dynamic url segments are prefixed with a colon. the router extracts and decodes these values and makes them available on `req.params` inside your controller.

```javascript
Route.get('/user/:id', 'app.IndexController@user');
Route.get('/admin/:entity/:id/edit', 'admin.AdminController@editForm');
```

```javascript
// controllers/app/IndexController.js
export default class IndexController {
  async user(req) {
    const user = await User.find(req.params.id);
    return json({ user });
  }
}
```

<a name="named-routes"></a>

## named routes

you give a route a name by chaining `.name()` after its definition. named routes allow you to generate urls programmatically without hardcoding paths.

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard').name('dashboard');
Route.get('/user/:id', 'app.IndexController@user').name('app.user.profile');
```

to generate a url from a named route you use the global `route()` helper.

```javascript
const dashboardUrl = route('dashboard');
const profileUrl = route('app.user.profile', { id: 42 });
```

<a name="route-groups"></a>

## route groups

groups allow you to share attributes like prefixes and middleware across a set of routes without repeating yourself on every definition.

<a name="prefix"></a>

### prefix

```javascript
Route.group({ prefix: '/faq' }, (faq) => {
  faq.get('/', 'app.IndexController@faq').name('app.faq');
  faq.get('/tos', 'app.IndexController@faq_tos').name('app.faq.tos');
  faq.get('/privacy', 'app.IndexController@faq_privacy').name('app.faq.privacy');
});
```

<a name="middleware"></a>

### middleware

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.get('/home', 'app.HomeController@home').name('app.home');
  auth.get('/library', 'app.IndexController@library').name('app.library');
});
```

<a name="combined-attributes"></a>

### combined attributes

prefix and middleware can be used together in a single group definition.

```javascript
Route.group({ prefix: '/admin', middleware: ['AuthMiddleware@requireAuth', 'AdminMiddleware@requireAdmin'] }, (admin) => {
  admin.get('/', 'admin.AdminController@index').name('admin.index');
  admin.get('/:entity', 'admin.AdminController@list').name('admin.list');
  admin.post('/:entity', 'admin.AdminController@store').name('admin.store');
});
```

groups can be nested. an inner group inherits all attributes of its parent and may add its own on top.

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.group({ prefix: '/api/preferences' }, (prefs) => {
    prefs.get('/', 'app.PreferencesController@index').name('api.preferences.index');
    prefs.put('/:key', 'app.PreferencesController@update').name('api.preferences.update');
  });
});
```

<a name="middleware-ordering"></a>

### middleware ordering

when a route sits inside one or more groups, the runtime middleware stack is `outer group middleware, inner group middleware, route level middleware, controller`. group middleware is prepended to the route's own handler chain at registration time, in declaration order, so nested groups run their parents' middleware before their own.

```javascript
Route.group({ middleware: [a] }, (g1) => {
  g1.group({ middleware: [b] }, (g2) => {
    g2.middleware(c).get('/path', 'Ctrl@index');
  });
});
// runtime stack for GET /path: [a, b, c, Ctrl@index]
```

middleware that returns a response without calling `next()` (such as the built in `RateLimiter` rejecting a request) short circuits the rest of the chain: the controller never runs and any later middleware never executes. this matters specifically when both a group and a route inside it apply a `RateLimiter`. the two limiters do not coordinate: each tracks its own window and its own count, whichever limit trips first wins, and reusing the same `RateLimiter` instance across both layers doubles the per request increment (so the effective limit becomes `max / 2`). see the [layered rate limiters](../security/security.md#layered-rate-limiters-group-plus-route) section in the security docs for the full interaction model.

<a name="domain-and-port-routing"></a>

## domain and port routing

dframework supports domain separated and port separated route registration. you can constrain routes to specific hosts subdomains or ports using either group attributes or chainable builder methods.

<a name="domain-routing"></a>

### domain routing

to restrict routes to a specific host or subdomain pass the `domain` option to `Route.group()` or call `Route.domain()`.

```javascript
Route.group({ domain: 'admin.example.com' }, (admin) => {
  admin.get('/dashboard', 'admin.AdminController@index').name('admin.dashboard');
});

Route.domain('api.example.com', (api) => {
  api.get('/v1/users', 'api.UserController@index').name('api.users');
});
```

<a name="port-routing"></a>

### port routing

you can also constrain routes to a specific listening port using the `port` option or `Route.port()`. port matching checks the host header or local server socket port.

```javascript
Route.group({ port: 8080 }, (metrics) => {
  metrics.get('/health', 'app.HealthController@check');
});

Route.port(9090, (internal) => {
  internal.get('/metrics', 'app.MetricsController@export');
});
```

<a name="chainable-builders"></a>

### chainable builders

`.domain()`, `.port()`, and `.middleware()` are fully chainable builders and can be composed in any order before registering endpoints or groups.

```javascript
Route.domain('shop.example.com').get('/cart', 'shop.CartController@index');

Route.domain('secure.example.com')
  .port(8443)
  .middleware('AuthMiddleware@requireAuth')
  .get('/data', 'app.DataController@show');

Route.domain('admin.example.com')
  .port(8080)
  .group({ prefix: '/v1' }, (v1) => {
    v1.get('/users', 'admin.UserController@index');
  });
```

<a name="parameterized-domains"></a>

### parameterized domains

domain definitions support dynamic parameter placeholders using either `:param` or `{param}` syntax. extracted domain parameters are automatically made available on `req.params` alongside path parameters.

```javascript
Route.domain(':subdomain.example.com', (tenant) => {
  tenant.get('/users/:id', 'app.TenantController@showUser');
});

Route.domain('{tenant}.myapp.com', (app) => {
  app.get('/settings', 'app.SettingsController@show');
});
```

```javascript
// GET http://acme.example.com/users/42
// req.params.subdomain === 'acme'
// req.params.id === '42'
```

<a name="overlapping-route-names-across-domains"></a>

### overlapping route names across domains

route names registered with `.name()` reside in a global lookup table. if two routes on different domains define the exact same `.name()` attribute (for example two separate `.name('dashboard')` calls), the later definition will overwrite the earlier entry in the `route()` helper map.

to avoid collisions when creating named routes across domain boundaries prefix your route names with the target domain or section name (for example `admin.dashboard` and `app.dashboard`).

```javascript
Route.domain('admin.example.com').get('/dashboard', 'AdminController@dash').name('admin.dashboard');
Route.domain('app.example.com').get('/dashboard', 'AppController@dash').name('app.dashboard');

// route('admin.dashboard') -> '//admin.example.com/dashboard'
// route('app.dashboard') -> '//app.example.com/dashboard'
```

when generating a url for a domain bound route using `route(name, params)`, the framework automatically returns a protocol relative absolute url (such as `//admin.example.com/dashboard`), filling in any domain parameters from the supplied `params` object.

<a name="inline-middleware-chains"></a>

## inline middleware chains

instead of a single controller string you can pass an array as the handler. the framework treats all entries before the last as middleware and the final entry as the controller.

```javascript
pair.post('/approve', ['AuthMiddleware@requireAuth', 'auth.AuthController@approvePost']).name('approve.post');
```

you can also apply middleware to a single route without a group by calling `.middleware()`.

```javascript
Route.middleware(authLimiter).post('/register', 'auth.AuthController@register');
```

<a name="route-modifiers"></a>

## route modifiers

every route definition returns a chainable object that exposes several modifiers to control framework behavior on that specific route.

<a name="csrf"></a>

### csrf

csrf verification is enabled by default on all `post`, `put`, and `delete` routes. you can disable it for a specific route by chaining `.csrf(false)`.

```javascript
Route.post('/magic/:token', 'auth.AuthController@magicConsume').csrf(false);
```

<a name="shield"></a>

### shield

when the shield security system is active it validates incoming mutation requests. you can opt a route out by chaining `.shield(false)`.

```javascript
Route.post('/magic/:token', 'auth.AuthController@magicConsume').shield(false);
```

<a name="log"></a>

### log

request and response logging is automatic in the `local` environment. you can force logging for a specific route in production by chaining `.log()`.

```javascript
Route.post('/api/playback/track', 'app.PlaybackController@get_track').name('api.playback.track').log();
```

<a name="views"></a>

### views

the `.views()` modifier tells the route compiler exactly which view templates this route renders. during the production build the compiler analyzes those templates to determine what session data locale detection and body parsing the compiled handler actually needs. providing explicit view names is an optimization hint that removes ambiguity from the static analysis step.

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard').name('dashboard').views(['app.home.index']);
```

<a name="profile"></a>

### profile

the `.profile()` modifier lets you override the behavior profile the compiler derives from its static analysis of the route. you pass an object whose keys replace the analyzed values. this is an advanced escape hatch for routes where the automatic analysis produces an incorrect or suboptimal compiled handler.

the available profile keys are:

- `needsSession`: boolean
- `needsPreviousUrl`: boolean
- `needsFlash`: boolean
- `needsBody`: boolean
- `needsLocale`: boolean
- `needsAntiPeeping`: boolean
- `hasCsrf`: boolean
- `hasShield`: boolean

```javascript
Route.get('/feed', 'app.FeedController@index').profile({ needsSession: true, needsFlash: true });
```

<a name="controller-string-syntax"></a>

## controller string syntax

controller strings follow the `directory.ControllerName@method` convention. the directory segment maps to a subdirectory inside `controllers/`. a controller at `controllers/app/IndexController.js` is referenced as `app.IndexController@index`. a controller at `controllers/auth/AuthController.js` is referenced as `auth.AuthController@register`.

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard');
Route.post('/register', 'auth.AuthController@register');
```

for controllers sitting directly in the `controllers/` root with no subdirectory you omit the directory prefix entirely.

```javascript
Route.get('/locale/:locale', 'LocaleController@set');
```

<a name="multiple-route-files"></a>

## multiple route files

you may split your routes across as many files as you like inside the `routes` directory. the framework loads all `.js` files it finds there automatically. the order of loading follows the filesystem sort order.

```javascript
// routes/web.js (http routes)
// routes/wire.js (socket routes)
```
