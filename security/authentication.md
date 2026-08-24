# authentication

- [authentication](#authentication)
  - [introduction](#introduction)
  - [configuration](#configuration)
  - [authenticating users](#authenticating-users)
    - [logging in](#logging-in)
    - [logging out](#logging-out)
    - [checking authentication state](#checking-authentication-state)
    - [retrieving the authenticated user](#retrieving-the-authenticated-user)
    - [retrieving user id](#retrieving-user-id)
  - [multi guard authentication](#multi-guard-authentication)
    - [defining guards](#defining-guards)
    - [using guards](#using-guards)
    - [flushing all guards](#flushing-all-guards)
  - [hashing](#hashing)
    - [standard hashing](#standard-hashing)
    - [deterministic fast hashing](#deterministic-fast-hashing)

<a name="introduction"></a>

## introduction

dframework makes implementing authentication extremely simple. the global `Auth` facade provides a simple, unified api for managing user sessions and authentication state across your application.

<a name="configuration"></a>

## configuration

authentication settings live in `config/auth.js`. by default, a single model authentication configuration points to `User`:

```javascript
// config/auth.js
export default {
  model: 'User'
};
```

the session cookie name can be customized in `config/app.js` via `sessionCookie` or the `SESSION_COOKIE` environment variable (defaults to `sid`):

```javascript
// config/app.js
export default {
  sessionCookie: Env.value('SESSION_COOKIE', 'sid'),
};
```

<a name="authenticating-users"></a>

## authenticating users

<a name="logging-in"></a>

### logging in

to log a user into your application, you may use the `login` method on the `Auth` facade. this method accepts the user model instance. you can also optionally pass additional session data as the second argument, and a boolean as the third argument to indicate if the session should be "permanent" (long lived).

> [!NOTE]
> when a user logs in via `Auth.login()`, their id is securely stored and the session identifier is regenerated automatically to mitigate session fixation attacks.

```javascript
import { Hash } from 'dframework';
import User from '../models/User.js';

export async function authenticate(req, res) {
  const user = await User.findByEmail('tarou@example.com');

  if (user && await Hash.verify('secret', user.password)) {
    // log the user in and set a long lived session cookie
    await Auth.login(user, { role: 'admin' }, true);
    
    return redirect('/dashboard');
  }

  return back('/login').withErrors({ email: 'invalid credentials' });
}
```

<a name="logging-out"></a>

### logging out

to log the user out of the default guard, use the `logout` method. this clears the default guard's session key while preserving other active guard sessions.

```javascript
await Auth.logout();
```

<a name="checking-authentication-state"></a>

### checking authentication state

to determine if the current request is authenticated, use the `check` method. it returns `true` if the user is logged in.

```javascript
if (Auth.check()) {
  // the user is logged in
}
```

<a name="retrieving-the-authenticated-user"></a>

### retrieving the authenticated user

you may access the authenticated user via the `user` method on the `Auth` facade. this returns the active user model instance, or `null` if the user is unauthenticated.

```javascript
const user = Auth.user();

if (user) {
  Log.info(`welcome back, ${user.name}`);
}
```

<a name="retrieving-user-id"></a>

### retrieving user id

to quickly retrieve the authenticated user id without accessing properties manually, use the `id` method:

```javascript
const userId = Auth.id();
```

<a name="multi-guard-authentication"></a>

## multi guard authentication

when your application needs independent authentication for multiple distinct entity types (for example, regular users and administrators), you can define guards in `config/auth.js`.

<a name="defining-guards"></a>

### defining guards

guards are registered under the `guards` object in `config/auth.js`. each guard specifies its model and optional custom session key (which defaults to `${guardName}Id`):

```javascript
// config/auth.js
export default {
  default: 'web',
  guards: {
    web: {
      model: 'User',
      sessionKey: 'userId'
    },
    admin: {
      model: 'Admin',
      sessionKey: 'adminId'
    }
  }
};
```

<a name="using-guards"></a>

### using guards

to perform authentication actions on a specific guard, use the `guard` method on the `Auth` facade:

```javascript
// check if an admin is logged in
if (Auth.guard('admin').check()) {
  const admin = Auth.guard('admin').user();
  const adminId = Auth.guard('admin').id();
}

// log in an admin (preserves active user sessions on other guards)
await Auth.guard('admin').login(adminUser);

// log out admin only
await Auth.guard('admin').logout();
```

all guard reads (`user()`, `check()`, `id()`) are completely synchronous during requests.

<a name="flushing-all-guards"></a>

### flushing all guards

to completely clear all active guard logins and wipe session data in one call, use `flush`:

```javascript
await Auth.flush();
```

<a name="hashing"></a>

## hashing

dframework provides a `Hash` facade which uses bcrypt for secure password hashing.

<a name="standard-hashing"></a>

### standard hashing

to hash a password, use the `make` method. it automatically generates a secure salt and applies 12 rounds of bcrypt hashing.

```javascript
import { Hash } from 'dframework';

const hashedPassword = await Hash.make('my-password');
```

to verify a plain text password against a hash, use the `verify` method.

```javascript
if (await Hash.verify('plain-text', hashedPassword)) {
  // passwords match
}
```

<a name="deterministic-fast-hashing"></a>

### deterministic fast hashing

occasionally, you may need to securely hash high entropy tokens (like api keys or personal access tokens) in a way that allows for exact database lookups. the `fast` method provides a deterministic sha256 hash using the server side `APP_KEY` as a pepper.

```javascript
// generating a token and saving its fast hash
const token = Hash.random(40);
const hashedToken = Hash.fast(token);

// querying the database for the exact hashed token
const record = await TokenModel.where('token', Hash.fast('provided-token')).first();
```
