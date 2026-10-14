# getting started

- [meet dframework](#meet-dframework)

  - [why dframework?](#why-dframework)

- [creating an application](#creating-an-application)

  - [installing the framework](#installing-the-framework)
  - [initializing a project](#initializing-a-project)
  - [booting the runtime](#booting-the-runtime)

- [initial configuration](#initial-configuration)

  - [environment configuration](#environment-configuration)

<a name="meet-dframework"></a>

## meet dframework

dframework represents an uncompromising approach to application development. it prioritizes architectural coherence over flexibility and assumes total control over the request lifecycle. there are no competing paradigms or interchangeable layers. the framework is the application.

<a name="why-dframework"></a>

### why dframework?

modern development increasingly relies on fragmented tooling layered abstractions and interchangeable architectures. this creates configuration fatigue and architectural inconsistency. the framework takes the opposite approach by enforcing a strict integrated environment.

<a name="creating-an-application"></a>

## creating an application

<a name="installing-the-framework"></a>

### installing the framework

to begin you must install the global framework binary. you do this by installing the @dstrn/framework package globally via npm. this provides the command line tool required to initialize and manage applications.

```bash
npm install -g @dstrn/framework
```

<a name="initializing-a-project"></a>

### initializing a project

once the binary is available on your system you can generate a new application. you use the init command and provide your application name. this constructs the exact directory structure required by the framework.

```bash
dstrn init myapp
cd myapp
```

<a name="booting-the-runtime"></a>

### serving the runtime

dframework enforces directory structure and specific booting mechanisms. applications are started via the provided command line tooling. the serve command initializes the application runtime compiles views and starts the internal server.

```bash
dstrn serve
```

<a name="initial-configuration"></a>

## initial configuration

<a name="environment-configuration"></a>

### environment configuration

internally the framework utilizes a central application instance. this instance controls everything from the router to the view engine and the database connection. it automatically detects the environment and configures the request pipeline accordingly.

when the application boots it registers the database connection initializes the logger and prepares the request handlers. during the booting process the framework also resolves the native socket router and the background queue workers.

> \[!NOTE]
> you never instantiate database connections, loggers, socket routers, or queue workers manually. they are resolved and managed exclusively by the framework runtime.

<br />

create a `.env` file in the project root. all values are parsed on startup.

the `config` directory houses modular configuration files that read from the environment using `Env.value()`
the following are basic configuration examples:

```javascript
// config/app.js
import { Env } from 'dframework';

export default {
  name: Env.value('APP_NAME', 'dframework'),
  env: Env.value('APP_ENV', 'production'),
  debug: Env.value('APP_DEBUG', false),
  url: Env.value('APP_URL', 'http://localhost'),
  port: Env.value('APP_PORT', 8250),
  locale: Env.value('APP_LOCALE', 'en'),
  key: Env.value('APP_KEY'),
  hashFields: (Env.value('HASH_FIELDS') || 'password').split(',').map(s => s.trim()).filter(Boolean),

  maxBodySize: Env.value('MAX_BODY_SIZE', 1),
  maxUploadSize: Env.value('MAX_UPLOAD_SIZE', 50),

  shield: Env.value('SHIELD_ENABLED', false),

  // tls: {
  //   cert: Env.value('TLS_CERT'),
  //   key: Env.value('TLS_KEY'),
  //   ca: Env.value('TLS_CA'),
  //   redirect: Env.value('TLS_REDIRECT', true),
  // },

  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
  },
};
```

```javascript
// config/auth.js
export default {
  model: 'User',

  // default: 'web',
  // guards: {
  //   web: {
  //     model: 'User',
  //     sessionKey: 'userId'
  //   },
  //   admin: {
  //     model: 'Admin',
  //     sessionKey: 'adminId'
  //   }
  // }
};
```

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
  connectionLimit: Env.value('DB_CONNECTION_LIMIT', '10'),
  maxIdle: Env.value('DB_MAX_IDLE', '5'),
  idleTimeout: Env.value('DB_IDLE_TIMEOUT', '60000'),
};
```

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

```javascript
// config/storage.js
import { Env } from 'dframework';

export default {
  disks: {
    local: {
      root: Env.value('STORAGE_ROOT', './storage'),
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

for full detail on configuration see the dedicated [configuration documentation](configuration.md)