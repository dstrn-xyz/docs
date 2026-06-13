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

<a name="inline-middleware-chains"></a>

## inline middleware chains

instead of a single controller string you can pass an array as the handler. the framework treats all entries before the last as middleware and the final entry as the controller.

```javascript
pair.post('/approve', ['AuthMiddleware@requireAuth', 'auth.AuthController@approvePost']).name('approve.post');
```

you can also apply middleware to a single route without a group by calling `.middleware()`.

```javascript
Route.middleware(authLimiter.middleware()).post('/register', 'auth.AuthController@register');
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
