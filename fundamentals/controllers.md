# controllers

- [controllers](#controllers)
  - [introduction](#introduction)
  - [writing controllers](#writing-controllers)
  - [directory structure and naming](#directory-structure-and-naming)
  - [optional arguments](#optional-arguments)
  - [returning responses](#returning-responses)
  - [accessing the request](#accessing-the-request)
    - [route parameters](#route-parameters)
    - [query strings](#query-strings)
    - [request body](#request-body)
    - [file uploads](#file-uploads)
    - [cookies](#cookies)
    - [pagination](#pagination)
  - [validation](#validation)
  - [using models and services](#using-models-and-services)
  - [private methods](#private-methods)
  - [hot reloading](#hot-reloading)

<a name="introduction"></a>

## introduction

controllers organize your request handling logic into focused classes. dframework automatically discovers and instantiates the correct controller class when a route is matched. you never construct controllers yourself.

<a name="writing-controllers"></a>

## writing controllers

a controller is a class with a default export. each method corresponds to a route action. all methods must be `async`.

```javascript
// controllers/app/IndexController.js
export default class IndexController {
  async index() {
    return render('auth.index');
  }

  async register() {
    return render('auth.register');
  }
}
```

you can generate a controller stub using the cli.

```bash
dstrn make:controller app/IndexController
```

<a name="directory-structure-and-naming"></a>

## directory structure and naming

controllers live inside the `controllers` directory and can be organized into subdirectories. the subdirectory and class name are reflected directly in the route definition string.

a controller at `controllers/app/HomeController.js` is referenced as `app.HomeController@method`. a controller at `controllers/auth/AuthController.js` is referenced as `auth.AuthController@method`. a controller at the root `controllers/LocaleController.js` is referenced as `LocaleController@method`.

```javascript
Route.get('/home', 'app.HomeController@home');
Route.post('/register', 'auth.AuthController@register');
Route.get('/locale/:locale', 'LocaleController@set');
```

<a name="optional-arguments"></a>

## optional arguments

controller methods receive `req` and `res` as arguments. both are optional. you only declare the ones your method actually uses. if your method does not need the request at all you omit it entirely.

```javascript
export default class IndexController {
  async index() {
    return render('auth.index');
  }

  async user(req) {
    const targetUser = await User.find(req.params.id);
    return render('app.user.profile', { targetUser });
  }
}
```

<a name="returning-responses"></a>

## returning responses

controllers return responses using the global helpers that the framework exposes. you never call methods on a response object directly.

```javascript
export default class PreferencesController {
  async index() {
    const preferences = await UserPreference.getAllForUser(Auth.user().id);
    return json({ success: true, preferences });
  }

  async update(req) {
    await UserPreference.setForUser(Auth.user().id, req.params.key, req.body.value);
    return json({ success: true });
  }

  async destroy() {
    return status(500).json({ success: false, message: 'failed to reset preference' });
  }
}
```

the available response helpers are `render()`, `json()`, `redirect()`, `back()`, `abort()`, and `status()`.

> \[!NOTE]
> the `abort(statusCode, message)` helper is generally preferred over manually setting `status()`. `abort()` is context aware: if the request was made via ajax, it automatically returns a json error payload. if it was a standard browser request, it renders the appropriate html error page (e.g. 404, 500).

```javascript
export default class UserController {
  async show(req) {
    const user = await User.find(req.params.id);
    if (!user) {
      // aborts with a 404, returning json for ajax requests or a 404 view for standard requests
      return abort(404, 'user not found');
    }
    return render('app.user.profile', { user });
  }
}
```

> \[!IMPORTANT]
> the `redirect(url, code = 302, force = false)` helper is strictly protected against open redirect vulnerabilities. it refuses to redirect to cross origin or protocol relative urls, throwing an error instead (except in local development). additionally, it is spa aware. if the request comes from the dSPA router, it returns a json instruction instead of a standard 302 header. setting the `force` boolean to `true` forces the spa router to perform a hard page reload on the client side.

```javascript
export default class AuthController {
  async logout(req) {
    await Auth.logout();
    // safely redirects to the home page, instructing the spa router to handle it
    return redirect('/home');
  }
}
```

<a name="accessing-the-request"></a>

## accessing the request

<a name="route-parameters"></a>

### route parameters

url segments prefixed with `:` in the route definition are extracted and available on `req.params`.

```javascript
export default class IndexController {
  async user(req) {
    const targetUser = await User.find(req.params.id);
    return render('app.user.profile', { targetUser });
  }
}
```

<a name="query-strings"></a>

### query strings

query string values are available on `req.query` as a plain object. the framework parses the query string lazily on first access.

```javascript
export default class SearchController {
  async index(req) {
    const { q, sort } = req.query;
    const results = await Track.where('title', q).get();
    return json({ results });
  }
}
```

<a name="request-body"></a>

### request body

the parsed request body is available on `req.body`. the framework automatically parses json and form encoded bodies on mutating requests before your controller method is called.

```javascript
export default class PreferencesController {
  async update(req) {
    const { key } = req.params;
    const { value } = req.body;
    await UserPreference.setForUser(Auth.user().id, key, value);
    return json({ success: true, key, value });
  }
}
```

<a name="file-uploads"></a>

### file uploads

when a request is submitted as `multipart/form-data` the framework parses the files and makes them available on `req.files`. each key on `req.files` is an array of file objects.

```javascript
import { Storage } from 'dframework';

export default class UserProfileController {
  async updateProfileImage(req) {
    const image = req.files.profileImage[0];
    const path = await Storage.disk('public').put(`user_assets/${Auth.user().id}/${image.originalFilename}`, image);
    return json({ success: true, path });
  }
}
```

<a name="cookies"></a>

### cookies

incoming cookies are available on `req.cookies` as a plain object. the framework parses the cookie header lazily on first access.

```javascript
export default class LocaleController {
  async set(req) {
    const preferred = req.cookies.locale;
    return json({ preferred });
  }
}
```

<a name="pagination"></a>

### pagination

dframework reads `req.page` automatically from the `page` query string parameter. you use `.paginate()` on any model query and the framework resolves the current page from the request context without any additional wiring.

```javascript
export default class AdminController {
  async list(req) {
    const paginatedResult = await User.paginate(20);
    return json(paginatedResult);
  }
}
```

the `.paginate(perPage)` method returns an object containing the records and metadata about the pagination state. you can access these properties directly:

- `data`: an array of model instances for the current page
- `total`: the total number of records across all pages
- `per_page`: the number of records requested per page
- `current_page`: the current page number
- `last_page`: the total number of pages
- `next_page_url`: the query string for the next page (e.g. `?page=3`) or `null`
- `prev_page_url`: the query string for the previous page or `null`

<a name="validation"></a>

## validation

the global `validate()` helper is available inside any controller method. you pass it either the request body object or the full `req` object when validating file uploads. see the [validation documentation](validation.md) for the complete list of available rules.

```javascript
export default class UserProfileController {
  async updateUsername(req) {
    const validation = validate(req.body, {
      username: 'required|string|min:3|max:20'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }

    // proceed with update
  }

  async updateProfileImage(req) {
    const validation = validate(req, {
      profileImage: 'file_required|file_size:8mb|file_max_count:1'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }

    // proceed with upload
  }
}
```

<a name="using-models-and-services"></a>

## using models and services

controllers commonly import models and services at the top of the file. because the framework hot reloads controllers in the `local` environment these imports are reevaluated on every file change.

```javascript
import Track from '../../models/Track.js';
import User from '../../models/User.js';

export default class HomeController {
  async home(req) {
    const leaderboard = await User.orderBy('current_rank', 'ASC').limit(5).get();
    const popularTracks = await Track.where('is_popular', 1).limit(5).get();

    return render('app.home.index', { leaderboard, popularTracks });
  }
}
```

<a name="private-methods"></a>

## private methods

controllers are plain classes so you can define non async helper methods alongside your route handlers. the framework only calls the method that matches the route definition. any other method on the class is completely private to your logic.

```javascript
export default class PreferencesController {
  async update(req) {
    const validation = this.validatePreference(req.params.key, req.body.value);
    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }
    await UserPreference.setForUser(Auth.user().id, req.params.key, req.body.value);
    return json({ success: true });
  }

  validatePreference(key, value) {
    return validate({ value }, { value: this.getValidationRules(key) });
  }

  getValidationRules(key) {
    const rules = {
      'audio.normalize_volume': 'boolean',
      'display.fps_counter': 'boolean',
    };
    return rules[key] || 'required';
  }
}
```

<a name="hot-reloading"></a>

## hot reloading

in the `local` environment the framework watches the `controllers` directory for file changes. when you save a controller file the framework reimports it automatically on the next request without restarting the server. you do not need to take any action to enable this behavior.
