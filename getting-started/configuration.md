# configuration

- [introduction](#introduction)

- [environment file](#environment-file)

- [config files](#config-files)

  - [config/app.js](#config-app)
  - [config/auth.js](#config-auth)
  - [config/database.js](#config-database)
  - [config/session.js](#config-session)
  - [config/storage.js](#config-storage)
  - [config/socket.js](#config-socket)
  - [config/native.js](#config-native)
  - [config/view.js](#config-view)

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
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost
APP_HOST=null
APP_PORT=8250
APP_LOCALE=en
APP_KEY=
APP_TRANSPORT=null
HASH_FIELDS=password

MAX_BODY_SIZE=1
MAX_UPLOAD_SIZE=50
TRUST_PROXY=false

SHIELD_ENABLED=false
SHIELD_DIFFICULTY=4
SHIELD_THRESHOLD=70

MAX_COMPILED_CACHE_SIZE=500
MAX_TEMPLATE_SOURCE_CACHE_SIZE=500

STATIC_FILE_CACHE_MAX_FILES=100
STATIC_FILE_CACHE_MAX_SIZE=1048576
STATIC_FILE_CACHE_MAX_TOTAL_BYTES=33554432

OPTIMIZE_CSS=true

TLS_CERT=null
TLS_KEY=null
TLS_CA=null
TLS_REDIRECT=true

QUEUE_MAX_WORKERS=4
QUEUE_TIMEOUT=60000
QUEUE_SHUTDOWN_TIMEOUT=30000

DB_DRIVER=mysql
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASS=
DB_NAME=dframework
DB_CONNECTION_LIMIT=10
DB_MAX_IDLE=5
DB_IDLE_TIMEOUT=60000
DB_QUERY_CACHE_MAX_ENTRIES=200
DB_QUERY_CACHE_MAX_ROWS=1000

SESSION_DRIVER=memory
SESSION_COOKIE=sid
SESSION_LIFETIME=120
SESSION_DOMAIN=null
SESSION_PATH=/
SESSION_SAME_SITE=Lax
SESSION_HTTP_ONLY=true
SESSION_SECURE=null
SESSION_MAX_SESSIONS=10000

STORAGE_ROOT=./storage
STORAGE_PUBLIC=./storage/public
```

the framework parses this file on startup. unquoted integer strings (such as `8250` or `-5`) are cast to safe integers. unquoted values of `true`, `false`, and `null` are cast to their boolean and null primitives. quoted values (wrapped in single or double quotes) are always preserved strictly as strings, protecting numeric passwords, hashes, tokens, or strings with leading zeros from being coerced. values are immediately available via `Env.value()` throughout your config files.

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
  host: Env.value('APP_HOST', null),
  port: Env.value('APP_PORT', 8250),
  locale: Env.value('APP_LOCALE', 'en'),
  basePath: Env.value('APP_BASE_PATH', null),
  langPath: Env.value('APP_LANG_PATH', null),
  key: Env.value('APP_KEY'),
  hashFields: (Env.value('HASH_FIELDS') || 'password').split(',').map(s => s.trim()).filter(Boolean),

  maxBodySize: Env.value('MAX_BODY_SIZE', 1),
  maxUploadSize: Env.value('MAX_UPLOAD_SIZE', 50),
  trustProxy: Env.value('TRUST_PROXY', false),
  transport: Env.value('APP_TRANSPORT', null),

  shield: Env.value('SHIELD_ENABLED', false),
  shield_difficulty: Env.value('SHIELD_DIFFICULTY', 4),
  shield_threshold: Env.value('SHIELD_THRESHOLD', 70),

  maxCompiledCacheSize: Env.value('MAX_COMPILED_CACHE_SIZE', 500),
  maxTemplateSourceCacheSize: Env.value('MAX_TEMPLATE_SOURCE_CACHE_SIZE', 500),

  staticFileCache: {
    maxFiles: Env.value('STATIC_FILE_CACHE_MAX_FILES', 100),
    maxSize: Env.value('STATIC_FILE_CACHE_MAX_SIZE', 1024 * 1024),
    maxTotalBytes: Env.value('STATIC_FILE_CACHE_MAX_TOTAL_BYTES', 32 * 1024 * 1024),
  },

  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
    safelist: [],
  },

  tls: {
    cert: Env.value('TLS_CERT', null),
    key: Env.value('TLS_KEY', null),
    ca: Env.value('TLS_CA', null),
    redirect: Env.value('TLS_REDIRECT', true),
  },

  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
    timeout: Env.value('QUEUE_TIMEOUT', 60000),
    shutdownTimeout: Env.value('QUEUE_SHUTDOWN_TIMEOUT', 30000),
  },
};
```

<a name="config-tuning"></a>

| key                             | env variable                        | accepted values                                | default                                      | description                                                                          |                                                                 |
| ------------------------------- | ----------------------------------- | ---------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------- |
| `name`                          | `APP_NAME`                          | string                                         | `'dframework'`                               | application name displayed in banners and logs                                       |                                                                 |
| `env`                           | `APP_ENV`                           | `'production' \| 'local'`                      | `'production'`                               | runtime environment mode                                                             |                                                                 |
| `debug`                         | `APP_DEBUG`                         | boolean (`true \| false`)                      | `false`                                      | enables detailed error diagnostic traces in local mode                               |                                                                 |
| `url`                           | `APP_URL`                           | string (valid url)                             | `'http://localhost'`                         | base url for links redirects and cli banners                                         |                                                                 |
| `host`                          | `APP_HOST`                          | string                                         | null (e.g. `'0.0.0.0'`, `'127.0.0.1'`, null) | `null`                                                                               | host address to bind http server (null binds to all interfaces) |
| `port`                          | `APP_PORT`                          | integer (1 to 65535)                           | `8250`                                       | tcp port number for the http server                                                  |                                                                 |
| `locale`                        | `APP_LOCALE`                        | string (iso language code e.g. `'en'`, `'fr'`) | `'en'`                                       | default language for view localization                                               |                                                                 |
| `basePath`                      | `APP_BASE_PATH`                     | string (directory path) \| null                | `process.cwd()`                              | custom root directory for controllers jobs and middlewares                           |                                                                 |
| `langPath`                      | `APP_LANG_PATH`                     | string (directory path) \| null                | `null` (resolves to `lang` under base path)  | custom directory containing localization dictionaries                                |                                                                 |
| `key`                           | `APP_KEY`                           | string (min 32 characters)                     | none (required in production)                | application secret used for session signing and hashing                              |                                                                 |
| `hashFields`                    | `HASH_FIELDS`                       | array of strings (comma separated in env)      | `['password']`                               | request parameters automatically hashed during database operations                   |                                                                 |
| `maxBodySize`                   | `MAX_BODY_SIZE`                     | integer (megabytes)                            | `1`                                          | maximum incoming request payload body size in mb                                     |                                                                 |
| `maxUploadSize`                 | `MAX_UPLOAD_SIZE`                   | integer (megabytes)                            | `50`                                         | maximum multipart file upload payload size in mb                                     |                                                                 |
| `trustProxy`                    | `TRUST_PROXY`                       | boolean (`true \| false`)                      | `false`                                      | trust x-forwarded-for and x-forwarded-proto proxy headers                            |                                                                 |
| `transport`                     | `APP_TRANSPORT`                     | `null \| 'http'`                               | `null`                                       | http server transport engine (null auto detects native dfws with node-http fallback) |                                                                 |
| `shield`                        | `SHIELD_ENABLED`                    | boolean (`true \| false`)                      | `false`                                      | enables antibot proof of work challenge protection                                   |                                                                 |
| `shield_difficulty`             | `SHIELD_DIFFICULTY`                 | integer (positive integer)                     | `4`                                          | leading hex zero difficulty for proof of work puzzle                                 |                                                                 |
| `shield_threshold`              | `SHIELD_THRESHOLD`                  | integer (0 to 100)                             | `70`                                         | proof of work pass score                                                             |                                                                 |
| `maxCompiledCacheSize`          | `MAX_COMPILED_CACHE_SIZE`           | integer (positive integer)                     | `500`                                        | maximum compiled view functions cached in memory                                     |                                                                 |
| `maxTemplateSourceCacheSize`    | `MAX_TEMPLATE_SOURCE_CACHE_SIZE`    | integer (positive integer)                     | `500`                                        | maximum raw view template files cached in memory                                     |                                                                 |
| `staticFileCache.maxFiles`      | `STATIC_FILE_CACHE_MAX_FILES`       | integer (positive integer)                     | `100`                                        | maximum number of static public assets held in memory                                |                                                                 |
| `staticFileCache.maxSize`       | `STATIC_FILE_CACHE_MAX_SIZE`        | integer (bytes)                                | `1048576` (1 mb)                             | maximum individual file size eligible for static cache                               |                                                                 |
| `staticFileCache.maxTotalBytes` | `STATIC_FILE_CACHE_MAX_TOTAL_BYTES` | integer (bytes)                                | `33554432` (32 mb)                           | total static file memory cache allocation limit                                      |                                                                 |
| `css.optimize`                  | `OPTIMIZE_CSS`                      | boolean (`true \| false`)                      | `true`                                       | enables css purging and minification during production builds                        |                                                                 |
| `css.safelist`                  | none                                | array of strings                               | `[]`                                         | css class names protected from purge removal                                         |                                                                 |
| `tls.cert`                      | `TLS_CERT`                          | string (file path) \| null                     | `null`                                       | path to tls ssl certificate file                                                     |                                                                 |
| `tls.key`                       | `TLS_KEY`                           | string (file path) \| null                     | `null`                                       | path to tls private key file                                                         |                                                                 |
| `tls.ca`                        | `TLS_CA`                            | string (file path) \| null                     | `null`                                       | path to intermediate certificate authority bundle file                               |                                                                 |
| `tls.redirect`                  | `TLS_REDIRECT`                      | boolean (`true \| false`)                      | `true`                                       | automatically redirect cleartext http traffic to https                               |                                                                 |
| `queue.maxWorkers`              | `QUEUE_MAX_WORKERS`                 | integer (positive integer)                     | `4`                                          | maximum concurrent background queue worker processes                                 |                                                                 |
| `queue.timeout`                 | `QUEUE_TIMEOUT`                     | integer (milliseconds)                         | `60000`                                      | execution timeout per queued job before termination                                  |                                                                 |
| `queue.shutdownTimeout`         | `QUEUE_SHUTDOWN_TIMEOUT`            | integer (milliseconds)                         | `30000`                                      | graceful shutdown drain timeout for queue workers                                    |                                                                 |

<a name="config-auth"></a>

### config/auth.js

the authentication config registers the models and guards the framework uses to resolve authenticated users.

#### single model authentication

for standard single model setups where one model handles authentication across the application:

```javascript
// config/auth.js
export default {
  model: 'User',
};
```

#### multi guard authentication

when your application uses multiple user entities (such as standard users and administrators), configure explicit guards:

```javascript
// config/auth.js
export default {
  default: 'web',
  guards: {
    web: {
      model: 'User',
      sessionKey: 'userId',
    },
    admin: {
      model: 'Admin',
      sessionKey: 'adminId',
    },
  },
};
```

#### http basic authentication

to enable basic auth headers on protected routes, define credentials under the `basic` key:

```javascript
// config/auth.js
export default {
  model: 'User',
  basic: {
    username: 'admin',
    password: 'password',
    realm: 'admin area',
  },
};
```

| key                        | env variable | accepted values                        | default                                            | description                                                 |
| -------------------------- | ------------ | -------------------------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| `model`                    | none         | string (model class name in `models/`) | `'User'`                                           | primary model class used for single guard authentication    |
| `default`                  | none         | string (guard name)                    | `'web'`                                            | default guard used when resolving authenticated users       |
| `guards`                   | none         | object of guard definitions            | `{ web: { model: 'User', sessionKey: 'userId' } }` | mapping of guard names to model and session key             |
| `guards.<name>.model`      | none         | string (model class name in `models/`) | `'User'`                                           | model class resolved for this guard                         |
| `guards.<name>.sessionKey` | none         | string (session key name)              | `'userId'` (for web/default) or `<name>Id`         | session key storing the authenticated user id               |
| `basic.username`           | none         | string                                 | none                                               | username required for http basic authentication             |
| `basic.password`           | none         | string                                 | none                                               | password required for http basic authentication             |
| `basic.realm`              | none         | string                                 | `'Restricted'`                                     | realm prompt string returned in 401 www-authenticate header |

<a name="config-database"></a>

### config/database.js

the database configuration defines the dialect driver, connection credentials, pool limits, and query cache parameters:

```javascript
// config/database.js
import { Env } from 'dframework';

export default {
  driver: Env.value('DB_DRIVER', 'mysql'),
  host: Env.value('DB_HOST', 'localhost'),
  port: Env.value('DB_PORT', 3306),
  user: Env.value('DB_USER', 'root'),
  pass: Env.value('DB_PASS', ''),
  name: Env.value('DB_NAME', 'dframework'),
  connectionLimit: Env.value('DB_CONNECTION_LIMIT', 10),
  maxIdle: Env.value('DB_MAX_IDLE', 5),
  idleTimeout: Env.value('DB_IDLE_TIMEOUT', 60000),
  queryCache: {
    maxEntries: Env.value('DB_QUERY_CACHE_MAX_ENTRIES', 200),
    maxRows: Env.value('DB_QUERY_CACHE_MAX_ROWS', 1000),
  },
};
```

| key                     | env variable                 | accepted values                  | default                                               | description                                          |
| ----------------------- | ---------------------------- | -------------------------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| `driver`                | `DB_DRIVER`                  | `'mysql' \| 'pgsql' \| 'sqlite'` | `'mysql'`                                             | database engine dialect driver                       |
| `host`                  | `DB_HOST`                    | string (hostname or ip)          | `'localhost'`                                         | database server host address (not used by sqlite)    |
| `port`                  | `DB_PORT`                    | integer (1 to 65535)             | `3306` (mysql) or `5432` (pgsql)                      | database server port                                 |
| `user`                  | `DB_USER`                    | string                           | `'root'` (mysql) or `'postgres'` (pgsql)              | database user account                                |
| `pass`                  | `DB_PASS`                    | string                           | `''`                                                  | database user password                               |
| `name`                  | `DB_NAME`                    | string                           | application database name, or `':memory:'` for sqlite | database name or sqlite database file path           |
| `connectionLimit`       | `DB_CONNECTION_LIMIT`        | integer (positive integer)       | `20` in production / `10` in local                    | maximum simultaneous database connections in pool    |
| `maxIdle`               | `DB_MAX_IDLE`                | integer (positive integer)       | `5`                                                   | maximum idle connections retained in pool            |
| `idleTimeout`           | `DB_IDLE_TIMEOUT`            | integer (milliseconds)           | `60000`                                               | duration before closing idle connections             |
| `queryCache.maxEntries` | `DB_QUERY_CACHE_MAX_ENTRIES` | integer (positive integer)       | `200`                                                 | maximum cached query plans in lru query cache        |
| `queryCache.maxRows`    | `DB_QUERY_CACHE_MAX_ROWS`    | integer (positive integer)       | `1000`                                                | maximum rows per result eligible for lru query cache |

<a name="config-session"></a>

### config/session.js

the session configuration defines the driver, cookie name, lifetime, and cookie parameters for user sessions:

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
  maxSessions: Env.value('SESSION_MAX_SESSIONS', 10000),
};
```

| key           | env variable           | accepted values                           | default    | description                                                       |
| ------------- | ---------------------- | ----------------------------------------- | ---------- | ----------------------------------------------------------------- |
| `driver`      | `SESSION_DRIVER`       | `'memory' \| 'database' \| 'stealth'`     | `'memory'` | session storage mechanism backend                                 |
| `cookie`      | `SESSION_COOKIE`       | string (valid cookie name)                | `'sid'`    | name of session id cookie sent to client                          |
| `lifetime`    | `SESSION_LIFETIME`     | integer (minutes)                         | `120`      | session lifespan in minutes before expiration                     |
| `domain`      | `SESSION_DOMAIN`       | string \| null                            | `null`     | domain scope for the session cookie                               |
| `path`        | `SESSION_PATH`         | string                                    | `'/'`      | path scope for the session cookie                                 |
| `same_site`   | `SESSION_SAME_SITE`    | `'Lax' \| 'Strict' \| 'None'`             | `'Lax'`    | samesite cookie policy                                            |
| `http_only`   | `SESSION_HTTP_ONLY`    | boolean (`true \| false`)                 | `true`     | prevents javascript access to session cookie                      |
| `secure`      | `SESSION_SECURE`       | boolean \| null (`true \| false \| null`) | `null`     | require https for session cookie (null auto-detects from request) |
| `maxSessions` | `SESSION_MAX_SESSIONS` | integer (positive integer)                | `10000`    | maximum active sessions retained by memory driver                 |

<a name="config-storage"></a>

### config/storage.js

the storage configuration defines the disks available through the storage facade:

```javascript
// config/storage.js
import { Env } from 'dframework';

export default {
  default: 'public',
  disks: {
    local: {
      root: Env.value('STORAGE_ROOT', './storage/local'),
    },
    public: {
      root: Env.value('STORAGE_PUBLIC', './storage/public'),
    },
    secure: {
      root: './storage/secure',
      encryptionKey: Env.value('APP_KEY'),
    },
  },
};
```

| key                          | env variable                           | accepted values                    | default                                    | description                                             |
| ---------------------------- | -------------------------------------- | ---------------------------------- | ------------------------------------------ | ------------------------------------------------------- |
| `default`                    | none                                   | string (disk name)                 | `'public'`                                 | default storage disk name used when omitted in disk()   |
| `disks`                      | none                                   | object of disk definitions         | `{ local: ..., public: ..., secure: ... }` | map of disk names to configuration objects              |
| `disks.<name>.root`          | `STORAGE_ROOT`, `STORAGE_PUBLIC`, etc. | string (directory path)            | none (required per disk)                   | local filesystem directory path for disk root           |
| `disks.<name>.encryptionKey` | `APP_KEY` or custom secret             | string (min 32 characters) \| null | `null`                                     | key for aes-256-gcm automatic encryption and decryption |

<a name="config-socket"></a>

### config/socket.js

the socket configuration controls real time websocket behavior and connection limits:

```javascript
// config/socket.js
import { Env } from 'dframework';

export default {
  maxPayload: Env.int('SOCKET_MAX_PAYLOAD', 1),
};
```

| key          | env variable         | accepted values    | default | description                                       |
| ------------ | -------------------- | ------------------ | ------- | ------------------------------------------------- |
| `maxPayload` | `SOCKET_MAX_PAYLOAD` | number (megabytes) | `1`     | maximum websocket frame payload size in megabytes |

<a name="config-native"></a>

### config/native.js

the native config defines everything related to building your application for native mobile and desktop targets. this file is loaded by the cli kernel in addition to `config/app.js`.

```javascript
// config/native.js
export default {
  name: null,
  index: null,
  fallback: 'native/static/index.html',
  identifier: 'com.dframework.app',
  version: '1.0.0',
  build: 1,
  icon: { source: 'native/assets/icon.png', background: '#ffffff', padding: 0 },
  splash: { enabled: true, background: '#0a0a0a', icon: 'native/assets/icon.png', iconSize: 0.3, fadeOut: 300 },
  theme: { statusBar: 'dark', primary: '#d3ac5f', background: '#0a0a0a', navigationBar: '#0a0a0a', safeArea: true },
  ios: { orientation: 'all', statusBarHidden: false },
  android: { orientation: 'unspecified', minSdk: 24, targetSdk: 34, permissions: ['INTERNET'] },
  desktop: { width: 1280, height: 720, resizable: true, minWidth: 800, minHeight: 600, titleBarStyle: 'default' },
};
```

| key                     | accepted values                              | default                           | description                                               |
| ----------------------- | -------------------------------------------- | --------------------------------- | --------------------------------------------------------- |
| `name`                  | string \| null                               | `null` (falls back to `app.name`) | display name on home screen and application window title  |
| `index`                 | string (url path) \| null                    | `null`                            | initial web route loaded by native container              |
| `fallback`              | string (file path)                           | `'native/static/index.html'`      | fallback static html file if remote server is unreachable |
| `identifier`            | string (reverse domain format)               | `'com.dframework.app'`            | unique bundle identifier for apple and android packages   |
| `version`               | string (semantic version)                    | `'1.0.0'`                         | user-facing application version string                    |
| `build`                 | integer (positive integer)                   | `1`                               | incremental build number / android version code           |
| `icon.source`           | string (file path)                           | `'native/assets/icon.png'`        | source icon image path (1024x1024 png)                    |
| `icon.background`       | string (hex color)                           | `'#ffffff'`                       | adaptive icon background color for android                |
| `icon.padding`          | number (0 to 1 float)                        | `0`                               | icon inset padding ratio                                  |
| `splash.enabled`        | boolean (`true \| false`)                    | `true`                            | enables native launch splash screen                       |
| `splash.background`     | string (hex color)                           | `'#0a0a0a'`                       | splash screen background color                            |
| `splash.icon`           | string (file path)                           | `'native/assets/icon.png'`        | splash screen center icon image path                      |
| `splash.iconSize`       | number (0 to 1 float)                        | `0.3`                             | splash icon scale relative to screen bounds               |
| `splash.fadeOut`        | integer (milliseconds)                       | `300`                             | splash screen dismissal transition duration in ms         |
| `theme.statusBar`       | `'light' \| 'dark'`                          | `'dark'`                          | native status bar icon appearance                         |
| `theme.primary`         | string (hex color)                           | `'#d3ac5f'`                       | primary brand accent color                                |
| `theme.background`      | string (hex color)                           | `'#0a0a0a'`                       | webview and window container background color             |
| `theme.navigationBar`   | string (hex color)                           | `'#0a0a0a'`                       | android bottom system navigation bar color                |
| `theme.safeArea`        | boolean (`true \| false`)                    | `true`                            | webview content respects device safe area insets          |
| `ios.orientation`       | `'all' \| 'portrait' \| 'landscape'`         | `'all'`                           | allowed device screen orientations on ios                 |
| `ios.statusBarHidden`   | boolean (`true \| false`)                    | `false`                           | hides the ios status bar                                  |
| `android.orientation`   | `'unspecified' \| 'portrait' \| 'landscape'` | `'unspecified'`                   | allowed screen orientation on android                     |
| `android.minSdk`        | integer (sdk api level)                      | `24`                              | minimum supported android sdk version                     |
| `android.targetSdk`     | integer (sdk api level)                      | `34`                              | target android sdk version                                |
| `android.permissions`   | array of strings                             | `['INTERNET']`                    | permissions declared in android manifest                  |
| `desktop.width`         | integer (pixels)                             | `1280`                            | default desktop application window width                  |
| `desktop.height`        | integer (pixels)                             | `720`                             | default desktop application window height                 |
| `desktop.minWidth`      | integer (pixels) \| undefined                | `800`                             | minimum allowed desktop window width                      |
| `desktop.minHeight`     | integer (pixels) \| undefined                | `600`                             | minimum allowed desktop window height                     |
| `desktop.resizable`     | boolean (`true \| false`)                    | `true`                            | allow user resizing of desktop window                     |
| `desktop.titleBarStyle` | `'default' \| 'hidden'`                      | `'default'`                       | desktop window title bar appearance                       |

<a name="config-view"></a>

### config/view.js

dframework supports an optional `config/view.js` configuration file for view engine settings.

```javascript
// config/view.js
import { Env } from 'dframework';

export default {
  maxYieldDepth: Env.value('VIEW_MAX_YIELD_DEPTH', 30),
};
```

| key | env variable | accepted values | default | description |
| --- | --- | --- | --- | --- |
| `maxYieldDepth` | `VIEW_MAX_YIELD_DEPTH` | integer (positive integer) | `30` | maximum recursion depth for nested `@yield` section resolution |


<a name="accessing-configuration-values"></a>

## accessing configuration values

dframework provides a global `Config` facade available in every context. you retrieve values using dot notation where the first segment is the config filename and the remainder is the key path within that file.

```javascript
const env = Config.get('app.env');
const dbName = Config.get('database.name');
const authModel = Config.get('auth.model', 'User');

if (Config.has('database.host')) {
  // configuration key is defined
}
```

the second argument to `Config.get` is an optional fallback returned when the key does not exist. if no fallback is provided and the key is missing the method returns `null`. use `Config.has` to check whether a configuration key is defined.

<a name="accessing-environment-values"></a>

## accessing environment values

inside your config files you use `Env.value()` to read from the parsed `.env` file. you can also use it anywhere you need a raw environment value without going through the config system.

```javascript
import { Env } from 'dframework';

const port = Env.value('APP_PORT', 8250);
const dbHost = Env.value('DB_HOST', 'localhost');
```

<a name="hot-reloading-configuration"></a>

## hot reloading configuration

in the `local` environment the framework watches the entire `config` directory for file changes. when you save a config file or `.env`, the watcher detects the change and restarts the server process so updated configuration values take effect immediately.
