# helpers

- [helpers](#helpers)
  - [introduction](#introduction)
  - [type checks](#type-checks)
  - [value checks](#value-checks)
  - [string and array utilities](#string-and-array-utilities)
  - [random and time](#random-and-time)
  - [html and security](#html-and-security)
  - [array helpers](#array-helpers)
  - [object helpers](#object-helpers)
  - [deep object access](#deep-object-access)
  - [url and asset helpers](#url-and-asset-helpers)
  - [response helpers](#response-helpers)
  - [routing helpers](#routing-helpers)
  - [validation](#validation)
  - [data sanitization](#data-sanitization)
  - [translation](#translation)
  - [facades](#facades)
    - [session](#session)
    - [auth](#auth)
    - [db](#db)
    - [job](#job)
    - [log](#log)

<a name="introduction"></a>

## introduction

dframework exposes a set of global helper functions available in every controller, middleware, model, view, and job without requiring any imports. these helpers cover type checking, data manipulation, url generation, response shortcuts, validation, translation, and facade access to core services.

all helpers are registered on the global scope when the application initializes.

<a name="type-checks"></a>

## type checks

| helper               | signature          | description                                                                                       |
| -------------------- | ------------------ | ------------------------------------------------------------------------------------------------- |
| `isUndefined(value)` | `(any) => boolean` | returns `true` if the value is `undefined`                                                        |
| `isNull(value)`      | `(any) => boolean` | returns `true` if the value is `null`                                                             |
| `isBoolean(value)`   | `(any) => boolean` | returns `true` if the value is a boolean                                                          |
| `isNumber(value)`    | `(any) => boolean` | returns `true` if the value is a number and not `NaN`                                             |
| `isInteger(value)`   | `(any) => boolean` | returns `true` if the value is an integer                                                         |
| `isString(value)`    | `(any) => boolean` | returns `true` if the value is a string                                                           |
| `isArray(value)`     | `(any) => boolean` | returns `true` if the value is an array                                                           |
| `isObject(value)`    | `(any) => boolean` | returns `true` if the value is a plain object (not `null`, not an array, constructor is `Object`) |
| `isFunction(value)`  | `(any) => boolean` | returns `true` if the value is a function                                                         |

```javascript
isString('hello');     // true
isNumber(42);          // true
isNumber(NaN);         // false
isObject({ a: 1 });    // true
isObject([1, 2]);      // false
isObject(null);        // false
```

<a name="value-checks"></a>

## value checks

**`isEmpty(value)`**

returns `true` if the value is considered empty. the following are considered empty: `undefined`, `null`, empty string `''`, empty array `[]`, and empty plain object `{}`.

```javascript
isEmpty('');           // true
isEmpty([]);           // true
isEmpty({});           // true
isEmpty(null);         // true
isEmpty(0);            // false
isEmpty('hello');      // false
isEmpty([1]);          // false
```

<a name="string-and-array-utilities"></a>

## string and array utilities

these helpers work polymorphically on both strings and arrays.

**`move(input, from, to)`**

moves an element from one index to another within a string or array. returns a new value without mutating the original.

```javascript
move([1, 2, 3, 4], 0, 2);   // [2, 3, 1, 4]
move('abcd', 0, 2);          // "bcad"
```

**`remove(input, index)`**

removes the element at the specified index from a string or array.

```javascript
remove([1, 2, 3], 1);        // [1, 3]
remove('hello', 1);           // "hllo"
```

**`replace(input, index, newItem)`**

replaces the element at the specified index with a new value.

```javascript
replace([1, 2, 3], 1, 9);    // [1, 9, 3]
replace('hello', 0, 'H');    // "Hello"
```

**`limit(input, maxLength)`**

truncates a string or array to the specified maximum length.

```javascript
limit('hello world', 5);     // "hello"
limit([1, 2, 3, 4, 5], 3);  // [1, 2, 3]
```

**`uniquify(input)`**

removes duplicate values from a string (by character) or array.

```javascript
uniquify([1, 2, 2, 3, 3]);  // [1, 2, 3]
uniquify('aabbcc');           // "abc"
```

<a name="random-and-time"></a>

## random and time

**`random(min, max)`**

generates a random integer between `min` and `max` (inclusive).

```javascript
random(1, 10);   // e.g. 7
random(0, 1);    // 0 or 1
```

**`now()`**

returns the current date and time as an iso 8601 string.

```javascript
now();   // "2026-06-05T02:30:00.000Z"
```

<a name="html-and-security"></a>

## html and security

**`escapeHtml(value)`**

escapes html special characters to prevent xss attacks. escapes `"`, `&`, `'`, `<`, and `>`. returns an empty string for `null` and `undefined`.

```javascript
escapeHtml('<script>alert("xss")</script>');
// "&lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;"

escapeHtml(null);   // ""
```

this function uses a single pass algorithm on large strings. the view engine's `{{ }}` syntax calls this function automatically.

**`emptyImage()`**

returns a data uri for a transparent 1x1 gif image. useful as a placeholder src attribute.

```javascript
emptyImage();
// "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
```

<a name="array-helpers"></a>

## array helpers

**`arrayFirst(arr, callback)`**

returns the first array element that satisfies the callback function. returns `undefined` if no element matches.

```javascript
arrayFirst([1, 2, 3, 4], n => n > 2);   // 3
```

**`arrayLast(arr, callback)`**

returns the last array element that satisfies the callback function.

```javascript
arrayLast([1, 2, 3, 4], n => n < 4);    // 3
```

**`pluck(arr, key)`**

extracts a single property from each object in an array.

```javascript
pluck([{ name: 'tarou' }, { name: 'satou' }], 'name');
// ['tarou', 'satou']
```

**`merge(...args)`**

deep merges multiple objects or arrays. arrays are concatenated. objects are recursively merged, with later values overriding earlier ones for conflicting keys.

```javascript
merge({ a: 1 }, { b: 2 });                    // { a: 1, b: 2 }
merge({ a: { x: 1 } }, { a: { y: 2 } });      // { a: { x: 1, y: 2 } }
merge([1, 2], [3, 4]);                        // [1, 2, 3, 4]
```

<a name="object-helpers"></a>

## object helpers

**`old(key, fallback)`**

retrieves flashed old input from the previous request. this is useful for repopulating form fields after a validation failure.

```javascript
old('email');                    // the previously submitted email value
old('name', 'default name');     // returns 'default name' if no old input exists
```

<a name="deep-object-access"></a>

## deep object access

these helpers provide safe access to deeply nested object properties using dot notation.

**`get(obj, path, defaultValue)`**

retrieves a nested value using a dot separated key path. returns `defaultValue` if the path does not exist.

```javascript
const config = { database: { host: 'localhost', port: 3306 } };

get(config, 'database.host');              // 'localhost'
get(config, 'database.name', 'mydb');      // 'mydb'
get(config, 'missing.key');                // undefined
```

**`set(obj, path, value)`**

sets a nested value using a dot separated key path. creates intermediate objects as needed. mutates and returns the original object.

```javascript
const config = {};
set(config, 'database.host', 'localhost');
// { database: { host: 'localhost' } }
```

**`has(obj, path)`**

returns `true` if the dot separated path exists in the object.

```javascript
has({ a: { b: 1 } }, 'a.b');    // true
has({ a: { b: 1 } }, 'a.c');    // false
```

**`forget(obj, path)`**

deletes a nested property by dot separated path. if the parent is an array, the element is spliced out. mutates and returns the original object.

```javascript
const data = { user: { name: 'satou', age: 30 } };
forget(data, 'user.age');
// { user: { name: 'satou' } }
```

<a name="url-and-asset-helpers"></a>

## url and asset helpers

**`asset(path)`**

generates a url for a static asset in the public directory. strips leading slashes and prepends a single `/`.

```javascript
asset('css/app.css');     // '/css/app.css'
asset('/js/main.js');     // '/js/main.js'
```

**`storage(path)`**

generates a url for a file in the storage directory. prepends `/storage/`.

```javascript
storage('avatars/user-1.jpg');     // '/storage/avatars/user-1.jpg'
storage(null);                     // ''
```

**`url(path, params)`**

generates a url with parameter substitution. replaces `:param` placeholders with the corresponding values from the params object. values are url encoded.

```javascript
url('/users/:id', { id: 42 });
// '/users/42'

url('/posts/:slug/comments/:commentId', { slug: 'hello-world', commentId: 7 });
// '/posts/hello-world/comments/7'
```

<a name="response-helpers"></a>

## response helpers

these helpers are shortcuts for calling methods on the current request context's response object. they can only be used inside a request context (controllers, middleware, etc.).

| helper                         | description                      |
| ------------------------------ | -------------------------------- |
| `render(view, data)`           | render a view template with data |
| `json(data)`                   | send a json response             |
| `redirect(url, code, force)`   | redirect to a url                |
| `back(fallback, code, force)`  | redirect to the previous url     |
| `abort(code, message)`         | throw an http error              |
| `status(code)`                 | set the response status code     |
| `cookie(name, value, options)` | set a response cookie            |

```javascript
export default class HomeController {
  async index() {
    return render('home', { title: 'welcome' });
  }

  async store(req) {
    await Item.create(req.body);
    return redirect('/items');
  }

  async api(req) {
    const items = await Item.all();
    return json({ items });
  }
}
```

<a name="routing-helpers"></a>

## routing helpers

**`route(name, params)`**

generates a url from a named route. substitutes `:param` placeholders with values from the params object.

```javascript
route('users.show', { id: 42 });
// '/users/42'
```

the `Route` facade must be initialized before using this helper. it is automatically available in all request contexts.

**`routeIs(name)`**

checks if the current request's route matches the given name. supports wildcard patterns with `*`.

```javascript
routeIs('dashboard');       // exact match
routeIs('admin.*');         // matches admin.users, admin.settings, etc.
```

<a name="validation"></a>

## validation

**`validate(dataOrReq, rules, messages)`**

validates data against a set of rules. accepts either a plain object or a request object (from which `body` and `files` are extracted).

```javascript
const v = validate(req, {
  email: 'required|email',
  password: 'required|string|min:8',
  age: 'nullable|integer|min:18'
});

if (v.fails()) {
  return json({ errors: v.errors() });
}
```

the validation result provides the following methods:

| method         | return type | description                    |                                          |
| -------------- | ----------- | ------------------------------ | ---------------------------------------- |
| `passes()`     | `boolean`   | `true` if all rules pass       |                                          |
| `fails()`      | `boolean`   | `true` if any rule fails       |                                          |
| `errors()`     | `object`    | all errors keyed by field name |                                          |
| `first(field)` | \`string    | null\`                         | first error message for a specific field |
| `all()`        | `object`    | alias for `errors()`           |                                          |

**available rules**

| rule           | description                                                         |
| -------------- | ------------------------------------------------------------------- |
| `required`     | value must not be empty                                             |
| `string`       | value must be a string                                              |
| `number`       | value must be numeric                                               |
| `integer`      | value must be an integer                                            |
| `boolean`      | value must be boolean-like (`true`, `false`, `1`, `0`, `yes`, `no`) |
| `array`        | value must be an array                                              |
| `email`        | value must be a valid email address                                 |
| `min:value`    | minimum length (string/array) or minimum value (number)             |
| `max:value`    | maximum length (string/array) or maximum value (number)             |
| `in:a,b,c`     | value must be one of the listed values                              |
| `not_in:a,b,c` | value must not be one of the listed values                          |
| `nullable`     | if the value is empty, skip all subsequent rules                    |

**file validation rules**

| rule                   | description                                            |
| ---------------------- | ------------------------------------------------------ |
| `file`                 | file must be present                                   |
| `file_required`        | file upload is required                                |
| `file_size:value`      | maximum file size (supports `kb`, `mb`, `gb` suffixes) |
| `file_min_size:value`  | minimum file size                                      |
| `file_mimes:type,type` | allowed mime types or file extensions                  |
| `file_max_count:n`     | maximum number of uploaded files                       |
| `file_min_count:n`     | minimum number of uploaded files                       |

```javascript
const v = validate(req, {
  avatar: 'file_required|file_size:5mb|file_mimes:jpg,png,webp',
  documents: 'file|file_max_count:10|file_size:20mb'
});
```

**custom error messages**

you can override the default error messages by passing a messages object keyed by `field.rule`.

```javascript
const v = validate(req, {
  email: 'required|email'
}, {
  'email.required': 'please enter your email address',
  'email.email': 'the email format is invalid'
});
```

<a name="data-sanitization"></a>

## data sanitization

**`sanitizeBody(data, fields)`**

sanitizes and casts raw input data based on field type definitions. this is useful for ensuring form data is cast to the correct types before processing.

```javascript
const cleaned = sanitizeBody(req.body, [
  { name: 'age', type: 'integer' },
  { name: 'score', type: 'number' },
  { name: 'active', type: 'boolean' },
  { name: 'tags', type: 'array' },
  { name: 'metadata', type: 'json' }
]);
```

| type      | behavior                                                                         |
| --------- | -------------------------------------------------------------------------------- |
| `number`  | parses strings to floats, passes through numbers                                 |
| `integer` | parses strings to integers, passes through integers                              |
| `boolean` | converts `'true'`, `'1'`, `'yes'` to `true`; `'false'`, `'0'`, `'no'` to `false` |
| `array`   | parses json arrays from strings, or splits comma-separated strings               |
| `json`    | parses json strings into objects                                                 |
| default   | passes through the value unchanged                                               |

fields with `null` or `undefined` values are omitted from the result.

<a name="translation"></a>

## translation

**`t(req, key, fallback)`**

retrieves a translated string for the current locale. see the [localization](/documentation/localization) documentation for full details.

```javascript
const welcome = await t(req, 'common.welcome');
const missing = await t(req, 'common.unknown', 'fallback text');
```

<a name="facades"></a>

## facades

dframework provides global facades for core services. these facades are available in the global scope and resolve to the current application instance's service.

<a name="session"></a>

### session

the `Session` facade provides access to the current request's session store.

| method                       | description                                                        |
| ---------------------------- | ------------------------------------------------------------------ |
| `Session.get(key, fallback)` | retrieve a value from the session                                  |
| `Session.set(key, value)`    | store a value in the session                                       |
| `Session.forget(key)`        | remove a value (or destroy the entire session if no key is passed) |
| `Session.flash(key, value)`  | store a value for only the next request                            |
| `Session.permanent(data)`    | store data with a long lived cookie (10 years)                     |

```javascript
await Session.set('theme', 'dark');
const theme = await Session.get('theme', 'light');
await Session.flash('status', 'saved successfully');
```

<a name="auth"></a>

### auth

the `Auth` facade provides access to the current user and authentication actions.

| method                              | description                                     |
| ----------------------------------- | ----------------------------------------------- |
| `Auth.user()`                       | returns the authenticated user object or `null` |
| `Auth.check()`                      | returns `true` if a user is authenticated       |
| `Auth.login(user, data, permanent)` | log in a user and create a session              |
| `Auth.logout()`                     | destroy the current session                     |

```javascript
if (Auth.check()) {
  const user = Auth.user();
}

await Auth.login(user, { role: 'admin' });
await Auth.logout();
```

`Auth.login()` accepts an optional third argument. when set to `true`, the session uses a long-lived cookie instead of a browser session cookie.

<a name="db"></a>

### db

the `DB` facade provides direct access to the database instance. it delegates all calls to the application's database connection.

```javascript
const users = await DB.table('users').where('active', true).get();
const count = await DB.table('orders').count();
await DB.raw('SELECT * FROM users WHERE id = ?', [1]);
```

<a name="job"></a>

### job

the `Job` facade dispatches background jobs to the worker pool. see the [jobs and queues](/documentation/jobs-and-queues) documentation for full details.

```javascript
Job.dispatch('SendEmailJob', { to: 'user@example.com', template: 'welcome' });
```

<a name="log"></a>

### log

the `Log` facade provides global access to the application logger. see the [logging and errors](/documentation/logging-and-errors) documentation for full details.

```javascript
Log.debug('processing started');
Log.info('user registered:', user.email);
Log.warn('rate limit approaching');
Log.error('payment failed:', err);
Log.table(rows, 'query results');
```
