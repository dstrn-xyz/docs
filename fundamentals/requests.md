# requests

- [introduction](#introduction)
- [the request context](#the-request-context)
- [request identity](#request-identity)
- [input and parameters](#input-and-parameters)
- [streaming requests](#streaming-requests)
- [headers](#headers)
- [csrf management](#csrf-management)
- [request tracking](#request-tracking)
  - [export sinks](#export-sinks)

<a name="introduction"></a>

## introduction

the `Request` class wraps the raw node http request to provide an api for interacting with incoming data. it parses the url, query strings, headers, and body payloads (json, form urlencoded, and multipart files).

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

// returns client ip address (honors X-Forwarded-For / X-Real-IP when app.trustProxy is enabled)
const clientIp = req.ip;

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

// retrieve unified input across params, body, and query (priority: params > body > query)
const name = req.input('name', 'guest');
const role = req.input('role');

// retrieve all merged inputs
const allInputs = req.all(); // or req.input()

// retrieve uploaded file instance (first item if array)
const avatar = req.file('avatar');

// check if a file was uploaded for the given key
if (req.hasFile('document')) {
  const doc = req.file('document');
}
```

as detailed in the [controllers documentation](controllers.md), `req.body` and `req.files` are populated automatically for mutating requests (`POST`, `PUT`, `PATCH`, `DELETE`) when csrf, shield, or body access is active on the route. if a route disables both csrf and shield and accesses the body through an indirect pattern not detected by static analysis, set `.profile({ needsBody: true })` as described in [routing](routing.md#profile). you can also use `req.file(key)` to retrieve the first file directly, or `req.hasFile(key)` to verify that an uploaded file is present.

<a name="streaming-requests"></a>

## streaming requests

to consume or pipe the raw incoming request payload directly without accumulating chunks in memory, you can obtain a readable stream:

```javascript
const stream = req.createReadableStream();
if (stream) {
  for await (const chunk of stream) {
    // process chunk
  }
}
```

<a name="headers"></a>

## headers

you can retrieve headers using the `header()` method, which is case insensitive.

```javascript
const userAgent = req.header('user-agent');
const referer = req.header('referrer'); // handles both referer and referrer
```

<a name="csrf-management"></a>

## csrf management

the request object handles generating and verifying csrf tokens.

> [!NOTE]
> the route compiler automatically inserts verification logic for all mutating routes (`POST`, `PUT`, `PATCH`, `DELETE`) unless explicitly disabled, so you rarely call these methods manually.

```javascript
// generate a new token valid for 6 hours
const token = req.csrfToken();

// verify a submitted token
const isValid = req.verifyCsrfToken(submittedToken);

// rotate the csrf secret and generate a fresh token (invalidates previous tokens)
const newToken = req.rotateCsrfToken();
```

<a name="request-tracking"></a>

## request tracking

every tracked request has an assigned correlation id accessible through `req.trackingId`. you can also capture a full snapshot of the request state at any point using the global `Tracker.capture()` facade (optionally accepting an error instance and explicit status code override, e.g. `Tracker.capture(err, 500)`), which records duration, headers, query, body, user, executed database queries, dispatched background jobs, and error details. calling `Tracker.capture()` exports the in progress snapshot to registered sinks without suppressing downstream error exports if an unhandled exception occurs later.

by default in production, tracking mode is `off`, introducing zero runtime overhead across the application. standard routes skip tracking initialization completely. when a specific route is marked with `.track()`, or when `Tracker.capture()` is called during a request, tracking activates lazily for that route only and streams to registered sinks without enabling tracking globally.

> [!NOTE]
> when tracking mode is `off` and a route is not declared with `.track()`, calling `Tracker.capture()` activates telemetry lazily from that point onward. to capture queries and background jobs executed from the beginning of the request, declare `.track()` on the route definition.

```javascript
export default class CheckoutController {
  process(req) {
    const trackingId = req.trackingId;
    const snapshot = Tracker.capture();
    return json({ trackingId, success: true });
  }
}
```

<a name="export-sinks"></a>

### export sinks

you can automatically stream request snapshots to local storage, external observability backends, or opentelemetry collectors using export sinks. sinks execute asynchronously in the background without blocking client responses. like other facades, `Tracker` is available globally without imports.

```javascript
// in memory ring buffer sink for testing and local inspection
const memSink = Tracker.sink('memory', { maxEntries: 100 });

// append snapshots as newline delimited json (ndjson) to storage
Tracker.sink('file', {
  path: 'storage/framework/tracking/requests.ndjson'
});

// stream snapshots as opentelemetry traces to an otlp collector
Tracker.sink('otlp', {
  endpoint: 'http://localhost:4318/v1/traces',
  serviceName: 'checkout-api'
});

// custom callback sink
Tracker.sink('observability', (snapshot) => {
  sendToTelemetry(snapshot);
});
```

sinks can also be configured declaratively in `config/tracking.js`:

```javascript
// config/tracking.js
import { Env } from 'dframework';

export default {
  mode: Env.value('TRACKING_MODE', 'off'),
  sinks: {
    file: {
      enabled: Env.value('TRACKING_FILE_ENABLED', false),
      path: Env.value('TRACKING_FILE_PATH', 'storage/framework/tracking/requests.ndjson')
    },
    otlp: {
      enabled: Env.value('TRACKING_OTLP_ENABLED', false),
      endpoint: Env.value('OTEL_EXPORTER_OTLP_ENDPOINT', 'http://localhost:4318/v1/traces'),
      serviceName: Env.value('OTEL_SERVICE_NAME', 'dframework-app')
    }
  }
};
```