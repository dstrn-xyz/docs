# responses

- [responses](#responses)

  - [introduction](#introduction)
  - [basic responses](#basic-responses)
  - [headers and cookies](#headers-and-cookies)
  - [json responses](#json-responses)
  - [rendering views](#rendering-views)
  - [streaming responses](#streaming-responses)
  - [redirects](#redirects)
  - [error responses](#error-responses)

<a name="introduction"></a>

## introduction

the `Response` class wraps the raw node http response. however, you almost never interact with the response object directly. instead, controllers return one of the globally available response helpers: `render()`, `json()`, `redirect()`, `back()`, `abort()`, `status()`, or `stream()`.

dframework detects the returned helper and finalizes the http response automatically.

<a name="basic-responses"></a>

## basic responses

the `status()` helper is used to set the http status code. it returns a chainable response object. you can chain `.json()` or `.send()` onto it.

```javascript
export default class WebhookController {
  async handle() {
    // responds with a 202 accepted and empty body
    return status(202).send('');
  }
}
```

<a name="headers-and-cookies"></a>

## headers and cookies

you can attach headers and cookies to the response before finalizing it. these methods are chainable.

```javascript
export default class DownloadController {
  async export() {
    const csv = await generateCsv();
    return status(200)
      .set('Content-Disposition', 'attachment; filename="export.csv"')
      .type('text/csv')
      .send(csv);
  }
}
```

to set a cookie, use the `.cookie()` method.

```javascript
export default class ThemeController {
  async update() {
    return status(200)
      .cookie('theme', 'dark', { httpOnly: false, path: '/', maxAge: 31536000 })
      .json({ success: true });
  }
}
```

<a name="json-responses"></a>

## json responses

the `json()` helper automatically sets the `Content-Type` to `application/json; charset=utf-8` and serializes the provided object.

```javascript
export default class ApiController {
  async user(req) {
    const user = await User.find(req.params.id);
    return json({ success: true, user });
  }
}
```

if validation errors were bound to the response via `.withErrors()`, the framework automatically injects them into the json payload. if the application is running in local development mode with the debugbar enabled, performance metrics are also automatically injected into the json payload.

<a name="rendering-views"></a>

## rendering views

the `render()` helper evaluates a view template and sends the resulting html. rendering is scheduled asynchronously, which allows you to chain data binding methods before the html is actually compiled.

```javascript
export default class ProfileController {
  async show() {
    const user = Auth.user();
    
    return render('app.profile.show', { user })
      .withData({ title: 'my profile' })
      .antiPeeping(false);
  }
}
```

if a form submission fails, you can flash the validation errors and the old input back to the view using `.withErrors()` and `.withInput()`. you can also flash custom key/value pairs using `.with(key, value)`.

```javascript
export default class SettingsController {
  async update(req) {
    const validation = validate(req.body, rules);
    
    if (validation.fails()) {
      return back().withErrors(validation.errors()).withInput(req.body).with('toast', 'settings could not be saved');
    }
    
    // ...
  }
}
```

<a name="streaming-responses"></a>

## streaming responses

the `stream(writer)` helper sends a response body in chunks with backpressure support. it receives an async function `(write, close)` where `write(chunk)` sends a chunk and returns a promise that resolves when the transport is ready for more data, and `close()` ends the response.

streaming is ideal for proxying large files (audio, video, documents) without buffering the entire payload in memory. set headers (content type, range, cache control) before calling `stream()`, just like any other response method.

dframework handles backpressure automatically. if the client or transport cannot keep up, `write()` will wait until the socket is writable again before resolving. the response pool is not released until `close()` is called, so long running streams do not interfere with object reuse.

```javascript
export default class StreamController {
  async audio(req) {
    const file = await AudioFile.find(req.params.id);
    if (!file) return abort(404);

    const upstream = await fetch(file.url, {
      headers: { range: req.headers['range'] }
    });

    return status(upstream.status)
      .set('accept-ranges', 'bytes')
      .set('content-range', upstream.headers.get('content-range'))
      .type('audio/mpeg')
      .stream(async (write, close) => {
        const reader = upstream.body.getReader();
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;
          await write(Buffer.from(value));
        }
        close();
      });
  }
}
```

`close()` must be called when you are done writing. if your writer function returns without calling `close()`, the framework will call it automatically for you. if the writer throws an error, the response is ended and the pool is released.

<a name="redirects"></a>

## redirects

the `redirect(url, code = 302, force = false)` helper creates a redirect response.

as noted in the [controllers documentation](controllers.md), dframework redirect helper is strictly protected against open redirect vulnerabilities. it refuses to redirect to cross origin or protocol relative urls, throwing an error instead (except in local development).

it is also spa aware. if the request was made by the client side router (`dSPAHttpRequest`), it returns a json instruction for the frontend router to execute instead of issuing a standard http 302. setting the `force` boolean to `true` forces the spa router to perform a hard page reload.

```javascript
export default class AuthController {
  async logout() {
    await Auth.logout();
    return redirect('/home');
  }
}
```

the `back()` helper redirects the user to their previous location based on the session's internal history tracking.

```javascript
export default class PostController {
  async destroy(req) {
    await Post.delete(req.params.id);
    return back('/fallback-url');
  }
}
```

<a name="error-responses"></a>

## error responses

the `abort(code, message)` helper immediately halts execution and returns an error response. it is context aware: if the request was made via ajax, it automatically returns a json error payload. if it was a standard browser request, it passes the error to the global error handler which renders the appropriate html error page (e.g. 404, 500).

```javascript
export default class FileController {
  async show(req) {
    const file = await File.find(req.params.id);
    
    if (!file) {
      return abort(404, 'file not found');
    }
    
    if (file.owner_id !== Auth.user().id) {
      return abort(403, 'unauthorized access');
    }
    
    return json({ file });
  }
}
```