# simulation and debugging

- [simulation and debugging](#simulation-and-debugging)
  - [introduction](#introduction)
  - [simulation mode](#simulation-mode)
    - [ios simulation](#ios-simulation)
    - [android simulation](#android-simulation)
    - [desktop simulation](#desktop-simulation)
  - [simulation vs production](#simulation-vs-production)
  - [debugging](#debugging)
    - [ios debugging](#ios-debugging)
    - [android debugging](#android-debugging)
    - [desktop debugging](#desktop-debugging)
  - [common issues](#common-issues)
    - [white screen on launch](#white-screen-on-launch)
    - [network requests failing](#network-requests-failing)
    - [static assets not loading](#static-assets-not-loading)
    - [javascript errors](#javascript-errors)
  - [log output](#log-output)

<a name="introduction"></a>

## introduction

the native simulation environment provides rapid iteration during development. unlike production builds, simulators connect directly to your local development server and support hot reloading.

<a name="simulation-mode"></a>

## simulation mode

simulation mode automatically injects your development server url into the native runtime. the framework starts your server if it is not already running.

<a name="ios-simulation"></a>

### ios simulation

```bash
dstrn simulate --ios
```

the framework searches for available ios simulators on your system. it prefers already booted devices and falls back to the latest available iphone model.

requirements:
- xcode installed with command line tools
- at least one ios simulator configured in xcode

the simulator displays console output from both the swift runtime and your javascript application. press `r` to reload the webview. press `q` to terminate.

<a name="android-simulation"></a>

### android simulation

```bash
dstrn simulate --android
```

the framework checks for connected physical devices first. if none are found, it searches for configured android virtual devices and launches the first one automatically.

requirements:
- android sdk installed
- platform tools available (adb)
- at least one avd configured in android studio

the emulator may take up to a minute to boot. the framework waits for the device to become ready before installing and launching your application.

android maps `localhost` to `10.0.2.2` automatically. your development server must be accessible at this address from within the emulator.

<a name="desktop-simulation"></a>

### desktop simulation

```bash
dstrn simulate --desktop
```

the desktop runtime opens a native window powered by the tauri webview. this behaves identically to a browser but with access to native system apis through the bridge.

requirements:
- rust toolchain installed
- tauri cli available

desktop simulation is the fastest iteration cycle. the window can be resized and responds to standard keyboard shortcuts. the console displays output from both the rust runtime and javascript environment.

<a name="simulation-vs-production"></a>

## simulation vs production

simulation and production modes differ in several critical ways:

| aspect | simulation | production |
|--------|------------|------------|
| url source | environment variable | configured url |
| server | local development | remote or static |
| reloading | instant via `r` key | full app restart |
| debugging | console output enabled | minimal logging |
| networking | localhost/10.0.2.2 | production endpoints |

the most important distinction is url resolution. simulators receive the development server url via `DSTRN_DEV_URL` environment variable. production builds use the url configured in [`config/native.js`](native-getting-started.md#url-configuration) or fall back to your app config.

if your production builds show a white screen, verify that the configured url is reachable and returns valid html. see [common issues](#white-screen-on-launch) for troubleshooting steps.

<a name="debugging"></a>

## debugging

the framework includes an integrated debug bar that provides realtime performance monitoring, network inspection, console logs, and dom analysis. the debug bar automatically activates in local development and can be toggled via the bottom bar interface.

key features:
- server timing and query profiling
- memory leak detection and cpu monitoring
- network request inspection with full headers and payloads
- console log aggregation with stack traces
- live dom inspector with visualization
- socket event monitoring
- component state tracking

the debug bar integrates with native webviews and provides the same experience across ios, android, and desktop platforms.

<a name="ios-debugging"></a>

### ios debugging

ios webview debugging works through safari developer tools:

1. enable develop menu in safari preferences
2. launch your app in the simulator
3. open safari develop menu
4. select simulator > your app > wkwebview

you now have full access to the javascript console, network inspector, and dom viewer. breakpoints and step debugging work as expected.

runtime logs from swift appear in the xcode console or terminal output where you launched the simulation command.

<a name="android-debugging"></a>

### android debugging

android webview debugging uses chrome devtools:

1. launch your app in the emulator or on a physical device
2. open chrome and navigate to `chrome://inspect`
3. find your app in the remote targets list
4. click inspect to open devtools

webview debugging is enabled automatically in debug builds. the framework injects network adapters that map localhost to the emulator's host ip.

use `adb logcat` to view native runtime logs:

```bash
adb logcat -s dstrn:V
```

<a name="desktop-debugging"></a>

### desktop debugging

desktop applications support devtools through the tauri webview. right click anywhere in the window and select inspect element.

console output appears both in the devtools console and the terminal where you launched the simulation. rust logs use the `[dstrn]` prefix.

<a name="common-issues"></a>

## common issues

<a name="white-screen-on-launch"></a>

### white screen on launch

symptom: the app launches but displays only a white screen with no content.

**in simulation mode:**

verify your development server is running and accessible:

```bash
curl http://localhost:825
```

for android, test the emulator mapping:

```bash
adb shell curl http://10.0.2.2:825
```

if the curl succeeds but the app shows white, check the browser console for javascript errors.

**in production builds:**

the most common cause is an unreachable or misconfigured url. production builds do not receive the development url automatically.

check your native configuration:

```javascript
// config/native.js
export default {
  index: 'https://your-actual-domain.com',
};
```

if `index` is null, verify that [`Config('app.url')`](configuration.md#app-config) and [`Config('app.port')`](configuration.md#app-config) point to your production server.

if your server requires network connectivity, consider implementing a [static fallback](native-getting-started.md#static-fallback) so the app can function offline.

<a name="network-requests-failing"></a>

### network requests failing

**cors errors in ios:**

ios requires explicit cors headers for cross origin requests. if your api runs on a different domain than your frontend, ensure your server includes proper cors headers.

**refused to connect in android:**

android blocks cleartext http by default. if your development or production server uses http instead of https, you must enable cleartext traffic in the manifest.

the framework allows cleartext by default in debug builds. production builds enforce https unless explicitly configured otherwise.

**localhost not reachable:**

on android, localhost refers to the emulator itself, not your development machine. all localhost references are automatically rewritten to `10.0.2.2` by the network polyfill.

on ios and desktop, localhost works as expected.

<a name="static-assets-not-loading"></a>

### static assets not loading

if your css or javascript files return 404 errors, verify the paths match your server configuration.

native webviews resolve relative urls against the page url. if your page loads from `https://api.example.com/` but your assets are served from `https://cdn.example.com/`, use absolute urls in your html.

the static fallback site resolves assets against the bundle root. paths like `/css/dstrn.css` work correctly in both remote and local mode.

<a name="javascript-errors"></a>

### javascript errors

check the javascript console for error messages and stack traces. the framework's error handler displays detailed debugging information in local mode.

common causes:
- undefined variables due to missing polyfills
- cors issues blocking network requests
- timing issues where dom elements are accessed before ready

the native bridge is available at `window.dstrnBridge` on all platforms. if your code depends on native features, verify the bridge exists before calling methods.

<a name="log-output"></a>

## log output

native runtimes emit standardized log messages:

```
[dstrn] loading remote: https://api.example.com
[dstrn] webview ready
[dstrn] load failed: network error
[dstrn] attempting fallback
```

simulation mode is more verbose than production. in production, only critical errors and state changes are logged.

you can filter native logs by platform:

```bash
# ios
xcrun simctl spawn booted log stream --predicate 'processImagePath contains "dframework"'

# android
adb logcat -s dstrn:V

# desktop
# logs appear in terminal where simulate was run
```
