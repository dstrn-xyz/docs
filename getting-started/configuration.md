# configuration

- [introduction](#introduction)

- [environment file](#environment-file)

- [config files](#config-files)

  - [config/app.js](#config-app)
  - [config/auth.js](#config-auth)
  - [config/native.js](#config-native)

- [accessing configuration values](#accessing-configuration-values)

- [accessing environment values](#accessing-environment-values)

- [hot reloading configuration](#hot-reloading-configuration)

<a name="introduction"></a>

## introduction

all application configuration lives in two places: the `.env` file at the project root and the `config` directory. the `.env` file holds environment specific values such as secrets and database credentials. the `config` directory holds structured javascript files that read from the environment and expose typed values to the rest of the application. the framework loads both automatically at boot.

<a name="environment-file"></a>

## environment file

the `.env` file sits at the root of your project and contains raw environment variable assignments. you copy the provided `.env.example` to `.env` and fill in your values. the `.env` file is never committed to version control.

```bash
APP_NAME=dframework
APP_KEY=
APP_LOCALE=en
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost
APP_PORT=0825

ANTI_PEEPING=false
SHIELD_ENABLED=false
SESSION_DRIVER=memory

QUEUE_MAX_WORKERS=4
OPTIMIZE_CSS=true

DB_HOST=localhost
DB_USER=root
DB_PASS=
DB_NAME=dframework
```

the framework parses this file on startup. string values of `true` and `false` are cast to booleans. numeric strings are cast to numbers. values are immediately available via `Env.value()` throughout your config files.

> \[!IMPORTANT]
> if you change the `.env` file while the application is running in the `local` environment the framework will restart the server entirely to pick up the new values.

<a name="config-files"></a>

## config files

each file in the `config` directory exports a default plain object and uses `Env.value()` to pull values from the environment. the filename becomes the configuration namespace used for all dot notation lookups throughout the application.

<a name="config-app"></a>

### config/app.js

the primary configuration file defines the core runtime behavior of the application.

```javascript
// config/app.js
import { Env } from 'dframework';

export default {
  name: Env.value('APP_NAME', 'dframework'),
  env: Env.value('APP_ENV', 'production'),
  debug: Env.value('APP_DEBUG', false),
  url: Env.value('APP_URL', 'http://localhost'),
  port: Env.value('APP_PORT', 825),
  locale: Env.value('APP_LOCALE', 'en'),
  key: Env.value('APP_KEY'),

  shield: Env.value('SHIELD_ENABLED', true),
  session_driver: Env.value('SESSION_DRIVER', 'memory'),

  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
  },

  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
  },

  database: {
    host: Env.value('DB_HOST', 'localhost'),
    user: Env.value('DB_USER', 'root'),
    pass: Env.value('DB_PASS', ''),
    name: Env.value('DB_NAME', 'dframework'),
  },
};
```

the `env` value must be either `local` or `production`. any other value will default to `production`. the `debug` flag is only active when `env` is `local`. the `key` value is used for session signing and route compilation fingerprinting and should be a long unique secret.

<a name="config-auth"></a>

### config/auth.js

the authentication config registers the model the framework uses to resolve authenticated users. by default it points to the `User` model.

```javascript
// config/auth.js
export default {
  model: 'User'
};
```

<a name="config-native"></a>

### config/native.js

the native config defines everything related to building your application for native mobile and desktop targets. this file is loaded by the cli kernel in addition to `config/app.js`.

```javascript
// config/native.js
export default {
  name: null,
  identifier: 'com.dframework.app',
  version: '1.0.0',
  build: 1,
  icon: { source: 'native/assets/icon.png', background: '#ffffff', padding: 0 },
  splash: { enabled: true, background: '#0a0a0a', icon: 'native/assets/icon.png' },
  theme: { statusBar: 'dark', primary: '#d3ac5f', background: '#0a0a0a' },
  ios: { orientation: 'all', statusBarHidden: false },
  android: { orientation: 'unspecified', minSdk: 24, targetSdk: 34, permissions: ['INTERNET'] },
  desktop: { width: 1280, height: 720, resizable: true },
};
```

<a name="accessing-configuration-values"></a>

## accessing configuration values

dframework provides a global `Config` facade available in every context. you retrieve values using dot notation where the first segment is the config filename and the remainder is the key path within that file.

```javascript
const env = Config.get('app.env');
const dbName = Config.get('app.database.name');
const authModel = Config.get('auth.model', 'User');
```

the second argument to `Config.get` is an optional fallback returned when the key does not exist. if no fallback is provided and the key is missing the method returns `null`.

<a name="accessing-environment-values"></a>

## accessing environment values

inside your config files you use `Env.value()` to read from the parsed `.env` file. you can also use it anywhere you need a raw environment value without going through the config system.

```javascript
import { Env } from 'dframework';

const port = Env.value('APP_PORT', 825);
const dbHost = Env.value('DB_HOST', 'localhost');
```

<a name="hot-reloading-configuration"></a>

## hot reloading configuration

in the `local` environment the framework watches the entire `config` directory for file changes. when you save a config file it is reimported and registered under the same namespace without restarting the server. this allows you to iterate on configuration values during development without interrupting the running process.
