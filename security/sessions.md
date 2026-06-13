# sessions

- [sessions](#sessions)
  - [introduction](#introduction)
  - [configuration](#configuration)
  - [interacting with the session](#interacting-with-the-session)
    - [retrieving data](#retrieving-data)
    - [storing data](#storing-data)
    - [flash data](#flash-data)
    - [deleting data](#deleting-data)
  - [session drivers](#session-drivers)
    - [stealth mode](#stealth-mode)

<a name="introduction"></a>

## introduction

since http driven applications are stateless, sessions provide a way to store information about the user across multiple requests. dframework provides an elegant `Session` facade available globally to interact with session data, regardless of the underlying storage driver.

<a name="configuration"></a>

## configuration

your application's session driver configuration is determined by the `app.sessionDriver` property. by default, the framework may use memory, database, or stealth drivers to persist session state.

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

if you need to store data permanently (using a ten year long lived cookie), use the `permanent` method.

```javascript
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
```

<a name="session-drivers"></a>

## session drivers

dframework abstracts away the complexity of session storage behind simple drivers.

- **memory**: stores sessions in ram. fast, but lost on server restart. suitable for local development.
- **database**: stores sessions in a dedicated database table. highly robust and scalable across multiple server instances.

<a name="stealth-mode"></a>

### stealth mode

dframework includes a highly unique `stealth` session driver. when stealth mode is enabled, session data is completely stateless on the server side. instead, the entire session payload is encrypted using aes-256-gcm and stored directly inside the user's secure http only session cookie.

this entirely eliminates the need for database lookups or memory overhead during session validation, allowing for extreme performance scaling while guaranteeing absolute data integrity and tamper detection.