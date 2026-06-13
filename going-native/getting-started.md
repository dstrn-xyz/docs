# native applications

- [native applications](#native-applications)
  - [introduction](#introduction)
  - [supported platforms](#supported-platforms)
  - [configuration](#configuration)
    - [basic setup](#basic-setup)
    - [application identity](#application-identity)
    - [url configuration](#url-configuration)
    - [static fallback](#static-fallback)
  - [development workflow](#development-workflow)
    - [simulation](#simulation)
    - [building](#building)
  - [project structure](#project-structure)

<a name="introduction"></a>

## introduction

dframework allows you to package your web application as native apps for ios, android, and desktop platforms. the native layer is a lightweight webview wrapper that provides device api access through a javascript bridge while keeping your application logic entirely web based.

unlike hybrid frameworks that require separate codebases or complex build configurations, dframework maintains a single source of truth. your routes, controllers, models, and views work identically across web and native environments.

<a name="supported-platforms"></a>

## supported platforms

the framework provides native runtimes for three platforms:

- **ios** - native swift application using wkwebview
- **android** - native kotlin application using android webview
- **desktop** - cross platform application using tauri (rust + webview)

each runtime includes automatic network adaptation, javascript polyfills, and a unified bridge api. no platform specific code is required in your application unless you need custom native plugins.

<a name="configuration"></a>

## configuration

<a name="basic-setup"></a>

### basic setup

native configuration lives in [`config/native.js`](config/native.js) at your project root. the framework merges your configuration with sensible defaults.

```javascript
export default {
  name: 'myapp',
  identifier: 'com.example.myapp',
  version: '1.0.0',
  build: 1,
};
```

if you omit the `name` field, the framework uses [`Config('app.name')`](configuration.md#app-config) as the fallback.

<a name="application-identity"></a>

### application identity

the `identifier` field must follow reverse domain notation and uniquely identifies your application on each platform. this value cannot be changed after distribution without creating a new app listing.

```javascript
export default {
  identifier: 'com.example.myapp',
  version: '2.1.0',
  build: 42,
};
```

the `version` string is user facing. the `build` number must increment with every release and is used by app stores to differentiate versions.

<a name="url-configuration"></a>

### url configuration

by default, native apps load content from [`Config('app.url')`](configuration.md#app-config) plus [`Config('app.port')`](configuration.md#app-config). you can override this behavior by setting the `index` field.

```javascript
export default {
  index: 'https://myapp.com',
};
```

when `index` is null, the framework constructs the url from your primary application config. this ensures that changing your api url automatically updates native builds without manual synchronization.

during simulation, the framework injects the development server url via environment variable, allowing you to test against localhost regardless of your production configuration.

<a name="static-fallback"></a>

### static fallback

you can bundle a static html site inside your native application as a fallback when the remote url is unreachable. this enables offline functionality or provides an error page with retry logic.

```javascript
export default {
  index: 'https://api.myapp.com',
  fallback: 'native/static/index.html',
};
```

the static site has full access to the bundled dframework css and javascript runtime. place your static files in the [`native/static/`](project-structure.md#native-static) directory. the framework copies this directory into the native bundle during build.

a minimal fallback page might display a network error with a reload button:

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="/css/dstrn.css">
  <title>offline</title>
</head>
<body class="bg-container flex-center" style="height: 100vh;">
  <div class="flex-column g-2 text-center p-2">
    <h1>no connection</h1>
    <p class="text-content-secondary">
      please check your internet connection and try again
    </p>
    <button onclick="location.reload()">retry</button>
  </div>
  <script src="/js/dstrn.js"></script>
</body>
</html>
```

the native runtime automatically attempts to load the fallback when the primary url fails. you can also build fully offline applications by setting `index` to null and providing a complete static implementation.

<a name="development-workflow"></a>

## development workflow

<a name="simulation"></a>

### simulation

use the `simulate` command to test your application in a native environment during development. the framework starts your server automatically if it is not already running.

```bash
dstrn simulate --ios
dstrn simulate --android
dstrn simulate --desktop
```

the simulator connects to your local development server. changes to your application code are immediately visible after refresh, just as they are in the browser.

press `r` to refresh the native app. press `q` to quit.

<a name="building"></a>

### building

the `build` command compiles a production ready binary for distribution. builds are written to [`native/builds/`](project-structure.md#native-builds) organized by platform.

```bash
dstrn build --target ios
dstrn build --target android
dstrn build --target desktop
```

production builds use the configured `index` url. if you have not set an explicit value, the framework uses your production api endpoint from [`config/app.js`](configuration.md#app-config).

builds require platform specific toolchains. see the [build and distribution documentation](native-build-distribution.md) for environment setup instructions.

<a name="project-structure"></a>

## project structure

native assets and configuration live in the `native` directory at your project root:

```
native/
├── assets/
│   └── icon.png       # application icon
├── static/            # optional static fallback site
│   ├── index.html
│   ├── css/
│   └── js/
└── builds/            # compiled binaries (gitignored)
    ├── ios/
    ├── android/
    └── desktop/
```

the framework generates the `builds` directory during compilation. you should exclude it from version control.

the `icon.png` file must be at least 1024x1024 pixels. the framework automatically generates all required sizes and formats for each platform during the build process.
