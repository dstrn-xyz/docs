# requests

- [introduction](#introduction)
- [the request context](#the-request-context)
- [request identity](#request-identity)
- [input and parameters](#input-and-parameters)
- [headers](#headers)
- [csrf management](#csrf-management)

<a name="introduction"></a>

## introduction

the `Request` class wraps the raw node http request to provide an api for interacting with incoming data. it parses the url, query strings, headers, and body payloads (json, formn urlencoded, and multipart files).

in controllers and middleware, the request instance is injected as the `req` argument.

<a name="the-request-context"></a>

## the request context

the framework utilizes `AsyncLocalStorage` to maintain request state across the entire asynchronous call stack. this allows you to access the current request globally without passing the `req` object down through every function.

```javascript
import { RequestContext } from 'dframework';

function getClientIp() {
  const req = RequestContext.get();
  return req ? req.headers['x-forwarded-for'] || req.socket.remoteAddress : null;
}
```

<a name="request-identity"></a>

## request identity

the request object provides several getters to determine the origin and format of the request.

```javascript
// returns true if the request comes from native builds
if (req.isNative) {
  return json({ success: true, native: true });
}

// returns true for XMLHttpRequest, dSPAHttpRequest, or application/json accept headers
if (req.isAjax) {
  return abort(422, 'validation failed');
}

// returns true specifically for the framework's client side spa router
if (req.isSPA) {
  return json({ component: 'Home' });
}
```

<a name="input-and-parameters"></a>

## input and parameters

the framework lazily parses input to avoid overhead on routes that don't need it.

```javascript
// parsed from the query string (e.g. ?search=term)
const query = req.query.search;

// parsed from the cookie header
const locale = req.cookies.locale;

// automatically extracts and ensures 'page' is an integer, defaulting to 1
const page = req.page;
```

as detailed in the [controllers documentation](controllers.md), `req.body` and `req.files` are populated automatically for mutating requests (`POST`, `PUT`, `PATCH`, `DELETE`) by the route compiler.

<a name="headers"></a>

## headers

you can retrieve headers using the `header()` or `get()` methods, which are case insensitive.

```javascript
const userAgent = req.header('user-agent');
const referer = req.header('referrer'); // handles both referer and referrer
```

<a name="csrf-management"></a>

## csrf management

the request object handles generating and verifying csrf tokens. the route compiler automatically inserts verification logic for all mutating routes unless explicitly disabled, so you rarely call these methods manually.

```javascript
// generate a new token valid for 6 hours
const token = req.csrfToken();

// verify a submitted token
const isValid = req.verifyCsrfToken(submittedToken);

// force rotation of the current token
const newToken = req.rotateCsrfToken();
```
