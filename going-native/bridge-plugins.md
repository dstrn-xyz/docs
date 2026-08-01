# native bridge and plugins

- [native bridge and plugins](#native-bridge-and-plugins)
  - [introduction](#introduction)
  - [the App.native facade](#the-appnative-facade)
    - [how method calls resolve](#how-method-calls-resolve)
    - [promises and errors](#promises-and-errors)
    - [the auto registered core plugins](#the-auto-registered-core-plugins)
  - [auth plugin](#auth-plugin)
  - [device plugin](#device-plugin)
  - [storage plugin](#storage-plugin)
  - [crypto plugin](#crypto-plugin)
  - [audio plugin](#audio-plugin)
  - [notifications plugin](#notifications-plugin)
  - [creating your own plugins](#creating-your-own-plugins)
    - [the four file convention](#the-four-file-convention)
    - [the javascript reference implementation](#the-javascript-reference-implementation)
    - [scaffolding a plugin](#scaffolding-a-plugin)
    - [writing the javascript source of truth](#writing-the-javascript-source-of-truth)
    - [ios implementation](#ios-implementation)
    - [android implementation](#android-implementation)
    - [desktop implementation](#desktop-implementation)
    - [fallback rules](#fallback-rules)
    - [end to end example a barcode scanner](#end-to-end-example-a-barcode-scanner)
  - [troubleshooting](#troubleshooting)

<a name="introduction"></a>

## introduction

the native bridge gives your javascript code access to device apis on ios, android, and desktop without leaving the web stack. everything is exposed through a single object, `App.native`, that works the same way on every platform.

each capability is a *plugin*. the framework ships six core plugins and lets you add as many of your own as you need. every plugin follows the same four file convention (one file per language), so a method you write once in javascript has matching implementations for swift, kotlin, and rust that you can fill in or skip.

this document covers both ends of that: the public `App.native.*` surface you call from your application code, and the four file layout you use when you want a capability to run natively on a platform.

<a name="the-appnative-facade"></a>

## the App.native facade

`App.native` is an object automatically attached to the global scope in both the browser and the native webview. each plugin is a property on that object, and each property exposes the plugin's methods directly.

```javascript
// every call is a promise
const pub = await App.native.crypto.generate();
await App.native.storage.set('theme', 'dark');
const id = await App.native.notifications.show('pairing complete', 'tap to open', 'pairing-1');
```

<a name="how-method-calls-resolve"></a>

### how method calls resolve

when you call `App.native.<plugin>.<method>(...)`, the bridge looks at the per plugin method map that the native runtime registered at boot:

- if the method is registered as a native method, the call is forwarded to the native layer (swift on ios, kotlin on android, the tauri command on desktop) and you receive the native result back as a promise.
- if the method is not registered as native, the call runs the javascript reference implementation directly. this happens for plugins that have no native file on that platform (desktop audio, desktop notifications, desktop crypto, desktop device, for example), and as a graceful fallback if the native layer fails to respond.

this decision is per method, not per plugin. you can register `audio.play` as native (so it uses avplayer on ios) and leave `audio.load` unregistered (so it uses the html audio element) on the same platform if you want to mix approaches.

<a name="promises-and-errors"></a>

### promises and errors

every method on `App.native.*` returns a promise. rejections carry a clear error message; methods that return a boolean (a successful storage write, for example) return `false` on failure rather than throwing. methods that cannot continue at all (notifications when the api is unavailable) throw.

use a single `await` in an `async` function for the simplest call:

```javascript
const granted = await App.native.notifications.requestPermission();
if (granted !== 'granted') {
  notify('enable notifications to receive pairing events', 'warning');
  return;
}
```

<a name="the-auto-registered-core-plugins"></a>

### the auto registered core plugins

the framework registers six core plugins automatically. each one is documented in full below:

- `auth` — local session id plus a generic key value store
- `device` — platform and model metadata
- `storage` — generic string key value store
- `crypto` — ecdsa p 256 sha 256 keypair, sign, verify, digest
- `audio` — single url playback with media session metadata
- `notifications` — show and cancel local notifications

<a name="auth-plugin"></a>

## auth plugin

the auth plugin gives every device a stable session id and a small key value store for persisting your own data. the web implementation uses `localStorage`; the ios implementation uses `UserDefaults`; the android implementation uses a dedicated `SharedPreferences` file. on every platform, the session key is namespaced under `dstrn.session.id` and the key value store is namespaced under `dstrn.db.<key>`, so your own keys cannot collide with the session.

#### `App.native.auth.setSession(id)`

stores `id` as the current device session. coerces to a string. returns `true` on success, `false` if the underlying storage throws (for example, in private browsing mode where localStorage writes are blocked).

```javascript
await App.native.auth.setSession(loginResponse.token);
```

#### `App.native.auth.getSession()`

returns the stored session id as a string, or `null` if none was ever stored or if the read threw.

```javascript
const id = await App.native.auth.getSession();
if (!id) redirect('/login');
```

#### `App.native.auth.clearSession()`

removes the stored session. returns `true` on success, `false` on a storage error.

```javascript
await App.native.auth.clearSession();
redirect('/');
```

#### `App.native.auth.db_get(key)`

reads the value previously stored under `dstrn.db.<key>`. returns the raw string the caller stored, or `null` if the key was never set.

```javascript
const lastDeviceId = await App.native.auth.db_get('last_device_id');
```

#### `App.native.auth.db_set(key, value)`

stores `value` under `dstrn.db.<key>`. `value` is stored as is, so json strings survive a round trip without being unwrapped or re encoded. returns `true` on success, `false` on a storage error.

```javascript
await App.native.auth.db_set('last_device_id', deviceId);
await App.native.auth.db_set('user_blob', JSON.stringify(user));
```

#### auth plugin notes

- `db_set` does not validate the value, so storing a parsed object will round trip as `[object Object]`. always store strings.
- the db namespace is intentionally separate from the storage plugin, which gives you a second independent key value store if you need one.
- on the web, both namespaces live in `localStorage`; clearing site data wipes them both.

<a name="device-plugin"></a>

## device plugin

the device plugin reports the platform and model of the current device. on ios it returns the device's `UIDevice.current.name` and model identifier; on android it returns `Build.MANUFACTURER + Build.MODEL`; on desktop it returns the hostname. on the web, the three methods return `navigator.userAgent`, `navigator.platform`, and the literal string `'web'`.

#### `App.native.device.name()`

returns a human readable device name. on ios, this is the name set in `Settings > General > About > Name` (for example, "jane's iphone"). on android, the manufacturer and model joined (for example, "google pixel 8"). on desktop, the hostname. on the web, the user agent string. always returns a string; falls back to `'unknown'` if the platform is unreachable.

```javascript
const name = await App.native.device.name();
els.deviceBadge.textContent = name;
```

#### `App.native.device.model()`

returns a model identifier without the manufacturer: `'iPhone'` or `'iPad'` on ios, the model name on android, the hostname on desktop. on the web, `navigator.platform`. always returns a string; falls back to `'unknown'`.

```javascript
const model = await App.native.device.model();
```

#### `App.native.device.platform()`

returns one of `'ios'`, `'android'`, `'desktop'`, or `'web'`. this is the simplest way to branch behavior by runtime.

```javascript
const platform = await App.native.device.platform();
if (platform === 'ios' || platform === 'android') {
  // enable native only features
}

<a name="storage-plugin"></a>

## storage plugin

a small, generic key value store for user preferences and ui state. on the web it is backed by `localStorage`; on ios by `UserDefaults`; on android by `SharedPreferences`; on desktop by a json file in the platform's application data directory. values are always strings — the plugin coerces non string values to strings on write and returns them unchanged on read.

the storage plugin and the auth db store are independent namespaces; use whichever fits the semantics of what you're storing.

#### `App.native.storage.set(key, value)`

stores `value` under `key`. returns `true` on success, `false` on failure. non string values are coerced via `String(value)`, so numbers, booleans, and json strings all work as long as you read them back consistently.

```javascript
await App.native.storage.set('theme', 'dark');
await App.native.storage.set('fontSize', 16);
await App.native.storage.set('lastSeen', String(Date.now()));
```

#### `App.native.storage.get(key)`

returns the stored string, or `null` if the key was never set. to distinguish a missing key from an empty string, check for `null` explicitly.

```javascript
const theme = await App.native.storage.get('theme');
if (theme === null) {
  // first run
}
```

#### `App.native.storage.remove(key)`

deletes the stored value. returns `true` if a value was removed, `false` otherwise (including when the key was already missing).

```javascript
await App.native.storage.remove('theme');
```

#### storage plugin notes

- there is no `keys`, `clear`, or `list` method. if you need those, persist them yourself as a json array under a fixed key.
- values larger than roughly five megabytes may fail on some platforms (`localStorage` has a per origin quota). store large blobs via `Storage.disk()` from the web framework instead.
- on ios and android, storage writes are synchronous but the bridge awaits them before resolving, so the promise pattern still works as expected.

<a name="crypto-plugin"></a>

## crypto plugin

the crypto plugin gives every device a stable ecdsa p 256 sha 256 keypair, plus helpers for signing, verifying, and digesting. every platform returns the same keys and signatures byte for byte, so a signature generated on ios verifies on android, the web, and desktop.

the keypair is persisted to a stable alias on every platform: `com.dstrn.crypto.main_key` in the ios keychain, `dstrn_crypto_main_key` in the android keystore, and the same alias in the web's indexeddb. calling `generate()` is idempotent — the first call creates and persists the keypair, every later call returns the same public key. the private key never leaves the secure store.

#### `App.native.crypto.generate()`

returns the device's public key as a base64 encoded raw uncompressed point (65 bytes, starting with `0x04`) and the algorithm string `'ECDSA-P256-SHA256'`. idempotent: calling `generate()` repeatedly always returns the same public key on the same device.

```javascript
const { publicKey, algorithm } = await App.native.crypto.generate();
console.log(algorithm); // 'ECDSA-P256-SHA256'
```

the returned object is `{ publicKey: string, algorithm: string }`.

#### `App.native.crypto.sign(data)`

signs `data` (base64 string) with the device's private key. returns `{ signature: string, algorithm: string }` where `signature` is a base64 encoded der asn.1 signature. rejects with a clear error if the keypair is unavailable (which should not happen after a successful `generate()`) or if `data` is not a valid base64 string.

```javascript
const payload = btoa(JSON.stringify({ op: 'pair', device: deviceName }));
const { signature } = await App.native.crypto.sign(payload);
```

the same payload and the same publicKey will verify on every platform.

#### `App.native.crypto.verify(data, signature, publicKey)`

verifies that `signature` is a valid signature of `data` under `publicKey`. all three arguments are base64 strings. returns `true` on success, `false` on failure (including malformed base64, malformed signatures, and tampered signatures).

```javascript
const ok = await App.native.crypto.verify(payload, signature, senderPublicKey);
if (!ok) throw new Error('signature mismatch');
```

#### `App.native.crypto.digest(data, algo?)`

hashes `data` (base64 string) with the requested algorithm. `algo` defaults to `'SHA-256'`. also accepts `'SHA-1'` and any algorithm the underlying platform supports (the web and ios accept `'SHA-256'` and `'SHA-1'`; android accepts the same in dashed or undashed form). returns the hash as a base64 string.

```javascript
const hash = await App.native.crypto.digest(btoa('abc'), 'SHA-256');
// hash === 'ungWv48Bz+pBQUDeXa4iI7ADYaOWG3Yr0FSxiyEyvg7gq9B3/L1lqg=='
```

#### `App.native.crypto.export()`

returns `{ publicKey, algorithm }` for the device's current keypair without signing anything. useful when you only need to share your public identity with a peer. behaves identically to `generate()` but is shorter to call.

```javascript
const { publicKey } = await App.native.crypto.export();
```

#### `App.native.crypto.import(publicKey)`

imports a foreign public key into the local keystore so subsequent `verify` calls can use it without reimporting. returns `true` on success. on the web, this is a no op that just validates the key shape and resolves to `true`.

```javascript
await App.native.crypto.import(peerPublicKey);
const ok = await App.native.crypto.verify(data, signature, peerPublicKey);
```

#### crypto plugin notes

- the public key is 65 raw bytes (0x04 prefix plus x and y coordinates), not a pem or x509 blob. this matches the format used by the ios `SecKeyCreateWithData` and android `ECPublicKeySpec` reconstruction paths, so cross platform verification works without transcoding.
- signatures are der encoded asn.1 (`SHA256withECDSA`), the format every platform produces natively. the byte length varies between roughly 64 and 72 depending on high entropy bits.
- on the web, the keypair is stored in indexeddb at `dstrn_crypto / keys / dstrn_crypto_main_key`. clearing site data wipes the keypair and the device will mint a new one on the next `generate()`. there is no way to migrate the keypair between devices.

<a name="audio-plugin"></a>

## audio plugin

single url audio playback with media session metadata, so the operating system can show title, artist, album, artwork, and playback controls on the lock screen and in the notification shade. a single shared player per page, so calling `load()` swaps the source. there is no queue and no multi url mode — one track at a time.

on ios the plugin uses `AVPlayer` and supports background playback and native now playing controls. on android it uses `MediaPlayer` plus `MediaSessionCompat` for the same effect. on the web and on desktop, the plugin falls back to `HTMLAudioElement` plus `MediaSession`, which is enough for in app playback and lock screen controls but cannot continue playing when the app is suspended.

#### `App.native.audio.load(url)`

loads `url` as the current source. only one url is supported; calling `load()` again replaces the previous track. returns `true` on success, `false` if the url is invalid or the audio element rejected.

```javascript
await App.native.audio.load(episodeUrl);
await App.native.audio.play();
```

#### `App.native.audio.play()`

starts playback of the currently loaded track. returns `true`. if the browser blocks autoplay (no user gesture yet), the underlying promise rejection is swallowed and `true` is returned; you should still test for user gesture by listening to the user event yourself before calling `play()`.

```javascript
button.addEventListener('click', async () => {
  await App.native.audio.play();
});
```

#### `App.native.audio.pause()`

pauses playback without resetting the current position. returns `true`.

#### `App.native.audio.resume()`

alias for `play()`. useful when the api is exposed as play/pause from the system media controls but your own code models it as paused/resumed.

#### `App.native.audio.seek(seconds)`

moves the playhead to `seconds` from the start of the track. accepts a number; fractional values are honored for frame accurate seeking. returns `true` if a track is loaded, `false` if no element exists yet (call `load` first).

```javascript
await App.native.audio.seek(120); // jump to two minutes in
```

#### `App.native.audio.stop()`

pauses playback and rewinds to the start of the track. returns `true`.

#### `App.native.audio.setRate(rate, preservePitch?)`

sets playback speed. `rate` is a number (`0.5` for half speed, `2` for double, etc). `preservePitch` defaults to `false`: when `false`, the pitch tracks the rate; when `true`, the pitch is held constant and only the speed changes. returns `true`.

```javascript
await App.native.audio.setRate(1.5, true); // 1.5x speed, normal pitch
```

#### `App.native.audio.setMetadata(title, artist, album, duration, artwork)`

publishes metadata to the system media session so the lock screen and notification shade can show what's playing. all arguments are optional except `title`. `duration` is in seconds and must be a number. `artwork` is a url string. returns `true` on success, `false` if the platform has no media session support.

```javascript
await App.native.audio.setMetadata(
  'episode 42: pairing flows',
  'dframework weekly',
  'season 2',
  1820,
  'https://cdn.example.com/art/episode-42.jpg'
);
```

#### audio plugin notes

- the same audio element is reused across calls. there is no per call player object.
- the plugin does not implement range request seeking across separate urls. one url, one source.
- `setRate` accepts any positive number; values outside roughly `[0.25, 4]` may not be honored by all platforms.

<a name="notifications-plugin"></a>

## notifications plugin

local notifications with a stable id per notification so you can update or cancel them after they're shown. on the web and desktop the plugin uses `window.Notification`; on ios it uses `UNUserNotificationCenter`; on android it uses `NotificationManagerCompat` with a runtime permission request on android 13+.

#### `App.native.notifications.requestPermission()`

returns the current permission state as a string: `'granted'`, `'denied'`, or `'prompt'`. if the state is `'prompt'`, the call will also prompt the user (which itself may resolve to `'granted'` or `'denied'`). on platforms without a notification api, returns `'denied'`.

```javascript
const status = await App.native.notifications.requestPermission();
if (status !== 'granted') {
  return notify('enable notifications in settings', 'warning');
}
```

#### `App.native.notifications.show(title, body?, id?)`

shows a local notification with `title` as the visible title and `body` as the visible body. `id` is optional; if you supply one, you can later `cancel(id)` to dismiss it. if you don't, the plugin generates an id like `'dstrn-0'`, `'dstrn-1'`, etc. and you cannot reliably cancel that specific one later.

returns the actual notification id used (your `id`, or the generated one). throws if the api is unavailable or if the permission has been denied.

```javascript
const id = await App.native.notifications.show(
  'pairing complete',
  'tap to open the dashboard',
  'pairing-success-1'
);
```

#### `App.native.notifications.cancel(id)`

dismisses the notification with the given `id`. returns `true` if the notification was found and dismissed, `false` if no notification with that id is known.

```javascript
await App.native.notifications.cancel('pairing-success-1');
```

#### notifications plugin notes

- permission must be granted before `show()` will succeed; otherwise it throws `permission denied`.
- notifications you do not give an explicit `id` to are not tracked for cancellation, by design. if you need to cancel later, always pass an `id`.
- the plugin does not schedule notifications for a future time. if you need that, schedule from your server using push notifications or a job queue.

<a name="creating-your-own-plugins"></a>

## creating your own plugins

every plugin is a folder with up to four files, one per language. the framework auto discovers any folder under `native/plugins/<name>/` in your project; there is no manifest, no schema, no registry, and no build wiring for you to edit. if the folder is there at build time, the plugin is shipped.

<a name="the-four-file-convention"></a>

### the four file convention

a plugin named `barcode` lives in `native/plugins/barcode/`:

```
native/plugins/barcode/
├── index.js       # required. javascript reference impl. source of truth for method names.
├── Plugin.swift   # optional. ios native impl. must export the same method names.
├── Plugin.kt      # optional. android native impl. must export the same method names.
└── Plugin.rs      # optional. desktop native impl. must export the same method names.
```

three rules govern this layout:

- `index.js` is **required**. the framework refuses to register a plugin without it, because the javascript impl is what makes the plugin usable on the web and serves as the fallback when a native impl is missing or fails. there is no exception.
- `Plugin.swift`, `Plugin.kt`, `Plugin.rs` are all optional. ship whichever native languages your team can support. missing files mean the plugin falls back to the javascript impl on that platform.
- method names are matched across languages by string equality. if `index.js` exports `barcode.scan`, then `Plugin.swift` must have a `case "scan":` branch, `Plugin.kt` must have a `"scan" ->` branch, and `Plugin.rs` must dispatch on `"scan"`. nothing else needs to align — parameter shapes and return shapes are each language's own concern, as long as the values you send across the bridge can be json encoded.

<a name="the-javascript-reference-implementation"></a>

### the javascript reference implementation

`index.js` is a plain module that exports a default object whose keys are method names and whose values are functions. the function body runs in the browser or webview and has access to all the normal browser globals: `window`, `localStorage`, `navigator`, `fetch`, `crypto.subtle`, `MediaSession`, `Notification`, `Audio`, and so on.

the methods run with the full power of the host environment. treat them like ordinary frontend javascript, just packaged into a plugin so the native side can override them per method per platform.

```javascript
// native/plugins/barcode/index.js
export default {
  scan() {
    return { ok: false, error: 'not implemented on this platform' };
  },
};
```

<a name="scaffolding-a-plugin"></a>

### scaffolding a plugin

the cli generates the four files with a working javascript impl and stub native files that you fill in:

```bash
dstrn make:plugin barcode
```

this creates `native/plugins/barcode/` with `index.js`, `Plugin.swift`, `Plugin.kt`, and `Plugin.rs`. after editing the methods you care about, the next `dstrn simulate` or `dstrn build` picks them up automatically.

<a name="writing-the-javascript-source-of-truth"></a>

### writing the javascript source of truth

because the javascript impl is the fallback for any missing native file, design it to work everywhere and treat native impls as performance or capability upgrades on top of it. if a feature genuinely cannot work on the web (camera capture, for example), return a clear error from the javascript impl:

```javascript
export default {
  async scan() {
    if (typeof navigator === 'undefined' || !navigator.mediaDevices) {
      throw new Error('barcode: camera unavailable in this environment');
    }
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
    // ... decode the stream ...
    stream.getTracks().forEach((t) => t.stop());
    return { value: '...', format: 'qr_code' };
  },
};
```

the plugin then upgrades to native implementations on ios and android while keeping the web behavior intact.

<a name="ios-implementation"></a>

### ios implementation

the framework generates a `Plugins.swift` file in the ios runtime project during build that calls into your per plugin `Plugin.swift`. inside your `Plugin.swift`, follow this shape:

```swift
import Foundation

struct BarcodePlugin {
    var bridge: Bridge?

    func handle(method: String, params: [String: Any], callId: String) {
        switch method {
        case "scan":
            scan(callId: callId)
        default:
            bridge?.reject(callId: callId, error: "barcode: unknown method \(method)")
        }
    }

    private func scan(callId: String) {
        // AVCaptureSession setup, delegate callbacks, etc.
        // call bridge?.resolve(callId: callId, result: [...]) on success
        // or bridge?.reject(callId: callId, error: "...") on failure
    }
}
```

the bridge will register `barcode.scan` as a native method at boot. the call returns whatever you pass to `resolve`.

<a name="android-implementation"></a>

### android implementation

the framework generates a `Plugins.kt` dispatcher that routes each method to your `Plugin.kt`. inside your `Plugin.kt`:

```kotlin
package com.dframework.native

import android.content.Context
import org.json.JSONObject

class BarcodePlugin(private val context: Context, private val bridge: Bridge) {
    fun handle(method: String, args: JSONObject, callId: String) {
        when (method) {
            "scan" -> scan(callId)
            else -> bridge.reject(callId, "barcode: unknown method $method")
        }
    }

    private fun scan(callId: String) {
        // CameraX or ML Kit setup
        // call bridge.resolve(callId, JSONObject().put("value", "...")) on success
        // or bridge.reject(callId, "...") on failure
    }
}
```

<a name="desktop-implementation"></a>

### desktop implementation

the framework generates a `plugins.rs` dispatcher. inside your `Plugin.rs`:

```rust
use serde_json::{json, Value};

pub fn dispatch(module: &str, method: &str, params: &Value) -> (bool, Value) {
    match (module, method) {
        ("barcode", "scan") => (true, json!({ "ok": false, "error": "no camera in desktop build" })),
        _ => (false, json!(format!("barcode: unknown method {}", method))),
    }
}
```

desktop webviews can use most browser apis (`fetch`, `crypto.subtle`, `localStorage`, etc), so a desktop plugin typically exists only when you need tauri specific apis like notifications or filesystem access that the browser does not have. if you don't ship a desktop file, the bridge falls back to the javascript impl automatically.

<a name="fallback-rules"></a>

### fallback rules

deciding which implementation runs is per method per platform:

- if `Plugin.swift` declares a `case "scan":` branch on ios, the bridge registers `barcode.scan` as native and every call from javascript is forwarded over the bridge.
- if you delete that branch (or the entire file), the bridge does not register `barcode.scan` as native on ios, so the javascript impl runs in the webview. this is what "fallback" means in practice: there is no platform code path that fails because a native file is missing.
- if the bridge registers a method as native but the native call rejects, the bridge surfaces the rejection to your promise; it does not silently fall back to javascript. design your native impls to either succeed or reject with a clear message.
- desktop audio, desktop notifications, desktop crypto, and desktop device all currently fall back to javascript — there is no `Plugin.rs` for those in the core framework. write one only if you need platform specific behavior on desktop.

<a name="end-to-end-example-a-barcode-scanner"></a>

### end to end example a barcode scanner

a complete minimal plugin that scans a barcode and returns its value, with a working web fallback and ios / android native stubs.

```javascript
// native/plugins/barcode/index.js
export default {
  async scan() {
    if (typeof navigator === 'undefined' || !navigator.mediaDevices) {
      throw new Error('barcode: camera unavailable');
    }
    const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
    try {
      // decoder logic, returns { value, format }
      return { value: 'placeholder-decoded-value', format: 'qr_code' };
    } finally {
      stream.getTracks().forEach((t) => t.stop());
    }
  },
};
```

```swift
// native/plugins/barcode/Plugin.swift
import AVFoundation

struct BarcodePlugin {
    var bridge: Bridge?

    func handle(method: String, params: [String: Any], callId: String) {
        switch method {
        case "scan": scan(callId: callId)
        default: bridge?.reject(callId: callId, error: "barcode: unknown method \(method)")
        }
    }

    private func scan(callId: String) {
        // AVCaptureMetadataOutput with .qr type, callback writes
        // bridge?.resolve(callId: callId, result: ["value": "...", "format": "qr_code"])
    }
}
```

```kotlin
// native/plugins/barcode/Plugin.kt
package com.dframework.native

import android.content.Context
import org.json.JSONObject

class BarcodePlugin(private val context: Context, private val bridge: Bridge) {
    fun handle(method: String, args: JSONObject, callId: String) {
        when (method) {
            "scan" -> scan(callId)
            else -> bridge.reject(callId, "barcode: unknown method $method")
        }
    }

    private fun scan(callId: String) {
        // CameraX + ML Kit, callback writes
        // bridge.resolve(callId, JSONObject().put("value", "...").put("format", "qr_code"))
    }
}
```

```rust
// native/plugins/barcode/Plugin.rs
use serde_json::{json, Value};

pub fn dispatch(module: &str, method: &str, params: &Value) -> (bool, Value) {
    match (module, method) {
        ("barcode", "scan") => (false, json!("barcode: camera not available on desktop")),
        _ => (false, json!(format!("barcode: unknown method {}", method))),
    }
}
```

calling this plugin from your application:

```javascript
try {
  const result = await App.native.barcode.scan();
  console.log(result.value, result.format);
} catch (e) {
  notify('could not scan: ' + e.message, 'error');
}
```

on ios and android, the native impl runs. on the web, the javascript impl runs. on desktop, the rust dispatch rejects with a clear error. one call site, four implementations.

<a name="troubleshooting"></a>

## troubleshooting

**the bridge never resolves a call.** check that the method is registered as native on the platform you're testing on. the framework ships `dstrn native:status` and `dstrn native:doctor` commands that report which methods are registered on each platform.

**a method returns a different value across platforms.** check that the json shape you return is the same on every platform. the bridge passes values through `JSONSerialization` on ios, `JSONObject` on android, and `serde_json` on desktop, so the wire format is the same; any difference is in the source language. compare the resolved value in your debugger.

**a plugin folder is not picked up.** make sure the folder name matches `<name>/` and that `index.js` exists. `dstrn native:doctor` reports discovered plugins and flags any that are missing `index.js`.

**native call times out.** the default timeout is thirty seconds, configurable via `native.timeout` in `config/native.js`. if your impl needs longer (large file uploads, network bound work), raise the timeout or break the work into smaller calls.
