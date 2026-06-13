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

to begin you must install the global framework binary. you do this by installing the dframework package globally via npm. this provides the command line tool required to initialize and manage applications.

```bash
npm install -g dframework
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

when the application boots it registers the database connection initializes the logger and prepares the request handlers. during the booting process the framework also resolves the native socket router and the background queue workers. you never instantiate these components manually. they are managed exclusively by the framework runtime.

<br />

create a `.env` file in the project root. all values are parsed on startup

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

> \[!IMPORTANT]
> applications are expected to follow framework structure exactly. configurations are available when they reinforce this philosophy.

for full detail on environment configuration see the dedicated [configuration documentation](configuration.md) 