# sessions

- [sessions](#sessions)
  - [introduction](#introduction)
  - [configuration](#configuration)
    - [session lifetime](#session-lifetime)
    - [cookie options](#cookie-options)
  - [interacting with the session](#interacting-with-the-session)
    - [retrieving data](#retrieving-data)
    - [storing data](#storing-data)
    - [flash data](#flash-data)
    - [deleting data](#deleting-data)
    - [session id and regeneration](#session-id-and-regeneration)
  - [session drivers](#session-drivers)
    - [stealth mode](#stealth-mode)

<a name="introduction"></a>

## introduction

since http driven applications are stateless, sessions provide a way to store information about the user across multiple requests. dframework provides an elegant `Session` facade available globally to interact with session data, regardless of the underlying storage driver.

<a name="configuration"></a>

## configuration

all session configuration is defined exclusively in `config/session.js`. the session driver is determined by `driver` (`memory`, `database`, or `stealth`), and the session cookie name is defined by `cookie` (defaults to `sid`).

<a name="session-lifetime"></a>

### session lifetime

the session duration determines how long a session remains active before expiration. you can configure the duration in minutes using `lifetime` (defaults to 120 minutes):

```javascript
// config/session.js
import { Env } from 'dframework';

export default {
  driver: Env.value('SESSION_DRIVER', 'memory'),
  cookie: Env.value('SESSION_COOKIE', 'sid'),
  lifetime: Env.value('SESSION_LIFETIME', 120),
  domain: Env.value('SESSION_DOMAIN', null),
  path: Env.value('SESSION_PATH', '/'),
  same_site: Env.value('SESSION_SAME_SITE', 'Lax'),
  http_only: Env.value('SESSION_HTTP_ONLY', true),
  secure: Env.value('SESSION_SECURE', null),
};
```

<a name="cookie-options"></a>

### cookie options

you can configure session cookie security attributes in `config/session.js`:

- `same_site`: controls cross site cookie transmission (defaults to `'Lax'`, accepted values: `'Lax'`, `'Strict'`, `'None'`)
- `secure`: forces cookie transmission over https (defaults to `null` which enables secure cookies automatically when app env is not local and disables them in local development, accepted values: `true`, `false`, `null`)
- `domain`: defines the cookie domain scope (defaults to `null` which restricts the cookie strictly to the current host origin without subdomains, accepted values: domain string such as `'.example.com'` or `null`)
- `path`: defines the cookie path scope (defaults to `'/'`, accepted values: path string such as `'/'`)
- `http_only`: prevents client side script access to the session cookie (defaults to `true`, accepted values: `true`, `false`)

<a name="interacting-with-the-session"></a>

## interacting with the session

<a name="retrieving-data"></a>

### retrieving data

to retrieve an item from the session, use the `get` method on the global `Session` facade. you may pass a default value as the second argument, which will be returned if the specified key does not exist.

```javascript
// retrieve a specific key
const value = await Session.get('key');

// retrieve a key with a default fallback
const name = await Session.get('name', 'guest');
```

<a name="storing-data"></a>

### storing data

to store data in the session, use the `set` method.

```javascript
await Session.set('key', 'value');
```

if you need to store data permanently (using a ten year long lived cookie), use the `permanent` method. you may pass key and value pairs or an object.

```javascript
await Session.permanent('role', 'admin');
await Session.permanent({ role: 'admin', accepted_terms: true });
```

<a name="flash-data"></a>

### flash data

sometimes you may wish to store items in the session for the next request only. you may do so using the `flash` method. data stored using this method will be available immediately and during the subsequent http request, after which it will be automatically deleted. this is highly useful for short lived status messages.

```javascript
await Session.flash('status', 'profile updated successfully');
```

<a name="deleting-data"></a>

### deleting data

to remove a piece of data from the session, use the `forget` method and pass the specific key. if you call `forget` without any arguments, the entire session will be destroyed and the cookie will be invalidated.

```javascript
// forget a single key
await Session.forget('key');

// destroy the entire session
await Session.forget();

// clear all session payload keys without invalidating authentication
await Session.flush();
```

> [!NOTE]
> `Session.flush()` removes all custom data keys stored in the active session payload without touching authentication guards or disconnecting active websocket connections. to revoke authenticated credentials and drop authorized websocket clients, call `Auth.logout()` or `Auth.flush()`.

<a name="session-id-and-regeneration"></a>

### session id and regeneration

to retrieve the active session identifier or set a custom one, use the `id` method.

```javascript
// retrieve the current session id
const sessionId = Session.id();
```

to prevent session fixation attacks when user privilege changes (such as during login or password reset), use the `regenerate` method. by default, it preserves existing session data while issuing a new session id and cookie. to clear all data and issue a fresh session, pass `false`.

```javascript
// regenerate session id while keeping existing session data
await Session.regenerate();

// regenerate session id and clear all existing session data
await Session.regenerate(false);
```

<a name="session-drivers"></a>

## session drivers

dframework abstracts away the complexity of session storage behind simple drivers.

- **memory**: stores sessions in ram. fast, but lost on server restart. suitable for local development.
- **database**: stores sessions in a dedicated database table. highly robust and scalable across multiple server instances.
- **stealth**: stores sessions entirely client side in encrypted cookies. zero server side storage or database queries required.

<a name="stealth-mode"></a>

### stealth mode

dframework includes a `stealth` session driver designed for privacy and zero state architectures.

when stealth mode is enabled (`SESSION_DRIVER=stealth`), session storage requires zero database queries, zero redis network roundtrips, and zero server memory consumption.

```javascript
// config/session.js
import { Env } from 'dframework';

export default {
  driver: Env.value('SESSION_DRIVER', 'stealth'),
  cookie: Env.value('SESSION_COOKIE', 'sid'),
  lifetime: Env.value('SESSION_LIFETIME', 120),
  same_site: 'Lax',
  http_only: true,
  secure: null
};
```

#### how stealth sessions operate

the entire session payload is serialized and encrypted with aes 256 gcm using your application secret key (`APP_KEY`). the resulting encrypted token string serves as the session identifier and is sent back to the browser in the configured http only cookie.

on incoming requests, the framework decrypts the cookie. if the cookie has been tampered with or modified by an attacker, decryption fails safely, logs a debug warning when `app.debug` is enabled, and initializes an empty guest session.

an expiration timestamp is embedded inside the encrypted payload. when saving session state, the driver refreshes the expiration window automatically or preserves future expiration dates (such as permanent sessions). expired sessions return null and are discarded.

because the entire session is stored inside an http cookie header, you must keep stored data minimal (strictly under 4kb to stay safely within web browser cookie size limits). stealth mode is ideal for user identifiers, auth tokens, and minimal preferences, but should not be used to store large arrays or bulky objects.

because no session rows or files exist on the server, there are no database tables to migrate and no scheduled garbage collection commands needed.