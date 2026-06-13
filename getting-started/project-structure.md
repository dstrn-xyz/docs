# project structure

- [introduction](#introduction)

- [the root directories](#the-root-directories)

  - [the config directory](#the-config-directory)
  - [the console directory](#the-console-directory)
  - [the controllers directory](#the-controllers-directory)
  - [the database directory](#the-database-directory)
  - [the lang directory](#the-lang-directory)
  - [the middleware directory](#the-middleware-directory)
  - [the models directory](#the-models-directory)
  - [the native directory](#the-native-directory)
  - [the public directory](#the-public-directory)
  - [the routes directory](#the-routes-directory)
  - [the storage directory](#the-storage-directory)
  - [the views directory](#the-views-directory)

<a name="introduction"></a>

## introduction

dframework enforces a deterministic application layout. you cannot configure or modify the default directory structure. this rigidity ensures that every application built with the framework is instantly familiar to any developer and that the internal engine can optimize loading and compilation paths aggressively.

> \[!NOTE]
> attempting to deviate from this directory structure will cause the framework engine to fail during the boot sequence or the view compilation phase.

<a name="the-root-directories"></a>

## the root directories

<a name="the-config-directory"></a>

### the config directory

the `config` directory holds all of your configuration files. the primary file is the application configuration where you define environment settings session drivers and security behaviors.

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
  antiPeeping: Env.value('ANTI_PEEPING', false),
  shield: Env.value('SHIELD_ENABLED', false),
  session_driver: Env.value('SESSION_DRIVER', 'memory'),
  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
  },
  database: {
    host: Env.value('DB_HOST', 'localhost'),
    user: Env.value('DB_USER', 'root'),
    pass: Env.value('DB_PASS', ''),
    name: Env.value('DB_NAME', 'dframework'),
  },
  storage: {
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
  }
};

```

```javascript
// config/native.js
export default {
  name: null,
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

<br />

### the console directory

background tasks and scheduling definitions are located in the `console` directory. the framework looks specifically for the schedule definition file here to configure recurring jobs and background workers. custom commands are also registered here.

```javascript
// console/Schedule.js
export default function (scheduler) {
  scheduler.command('CleanupExpiredSessions').daily();
}
```

<a name="the-controllers-directory"></a>

### the controllers directory

the `controllers` directory contains the classes that handle your incoming request logic. controllers organize your behavior into simple async methods that are invoked automatically by the router. there is no base controller class to extend. arguments like the request and response objects are entirely optional if your logic does not require them.

```javascript
// controllers/IndexController.js
export default class IndexController {
  async index() {
    return render('index');
  }
}
```

<a name="the-database-directory"></a>

### the database directory

the `database` directory holds all of your database related files including your migrations and seeders. the framework utilizes this directory when executing schema building commands allowing you to track database changes across environments.

```javascript
// database/migrations/0001_create_users_table.js
export async function up() {
  await Schema.create('users', table => {
    table.increments();
    table.string('email').unique();
    table.string('password');
    table.timestamps();
  });
}

export async function down() {
  await Schema.dropIfExists('users');
}
```

```javascript
// database/seeders/DatabaseSeeder.js
export async function run() {
  await table('users').insert({
    email: 'admin@example.com',
    password: 'password' // passwords are automatically hashed
  });
}
```

<a name="the-lang-directory"></a>

### the lang directory

localization strings and translation dictionaries are stored in the `lang` directory. these are typically stored as json files inside locale specific folders. the framework automatically loads these files to provide multilingual support across your application and view templates.

```json
// lang/en/common.json
{
  "welcome": "welcome to dframework",
  "login": "log in to your account"
}
```

<a name="the-middleware-directory"></a>

### the middleware directory

the `middleware` directory contains classes that filter and inspect http requests entering your application. you define custom logic here to enforce authentication rate limiting and request manipulation before it reaches your controllers.

```javascript
// middleware/EnsureAuth.js
export default class EnsureAuth {
  handle(req, next) {
    if (!Auth.check()) return abort(401);
    return next(req);
  }
}
```

<a name="the-models-directory"></a>

### the models directory

your active record entities reside in the `models` directory. the framework relies on this exact location to dynamically resolve models particularly during authentication flows and implicit route model binding. models represent the single source of truth for your database schema abstractions.

```javascript
// models/User.js
export default class User extends Model {
  // model definition
}
```

<a name="the-native-directory"></a>

### the native directory

the `native` directory contains code specifically required for integrating the web application into native mobile and desktop environments. the framework handles native compilation and asset injection utilizing this directory to bridge web views with native device capabilities.

<a name="the-public-directory"></a>

### the public directory

static assets and public facing files belong in the `public` directory. the internal web server is capable of serving these files directly during local development. in production environments this directory acts as the root for your web server configuration.

<a name="the-routes-directory"></a>

### the routes directory

the `routes` directory is the entry point for all incoming requests. the framework automatically reads and registers all routing definitions found within this folder. you define your web endpoints and application logic mappings here. there is no need to manually import these files elsewhere as the framework handles discovery during the boot process.

```javascript
// routes/web.js
Route.get('/', 'HomeController@index');
```

<a name="the-storage-directory"></a>

### the storage directory

the `storage` directory is reserved for application generated files. this includes cached views temporary hot reloading data and user uploaded content. the framework automatically manages the internal structure of this directory and maintains a public symbolic link for accessible uploads.

<a name="the-views-directory"></a>

### the views directory

the `views` directory contains all frontend templates. the internal view compiler statically analyzes and optimizes these templates during the production build pipeline. you cannot place templates outside this directory as the compilation engine specifically targets this path.

```blade
<!-- views/welcome.blade -->
@extends('layouts.app')

@section('content')
  <div>welcome to dframework</div>
@endsection
```