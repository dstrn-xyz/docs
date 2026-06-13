# authentication

- [authentication](#authentication)
  - [introduction](#introduction)
  - [authenticating users](#authenticating-users)
    - [logging in](#logging-in)
    - [logging out](#logging-out)
    - [checking authentication state](#checking-authentication-state)
    - [retrieving the authenticated user](#retrieving-the-authenticated-user)
  - [hashing](#hashing)
    - [standard hashing](#standard-hashing)
    - [deterministic fast hashing](#deterministic-fast-hashing)

<a name="introduction"></a>

## introduction

dframework makes implementing authentication extremely simple. the global `Auth` facade provides a simple, unified api for managing user sessions and authentication state across your application.

<a name="authenticating-users"></a>

## authenticating users

<a name="logging-in"></a>

### logging in

to log a user into your application, you may use the `login` method on the `Auth` facade. this method accepts the user model instance. you can also optionally pass additional session data as the second argument, and a boolean as the third argument to indicate if the session should be "permanent" (long lived).

when the user is logged in, their id is securely stored in the session and the session identifier is regenerated to prevent session fixation attacks.

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

to log the user out, use the `logout` method. this clears the user's session data entirely.

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
