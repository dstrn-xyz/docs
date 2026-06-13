# native bridge and plugins

- [native bridge and plugins](#native-bridge-and-plugins)
  - [introduction](#introduction)
  - [the native bridge](#the-native-bridge)
    - [bridge architecture](#bridge-architecture)
    - [calling native methods](#calling-native-methods)
    - [receiving native events](#receiving-native-events)
  - [available apis](#available-apis)
    - [storage](#storage)
    - [clipboard](#clipboard)
    - [haptics](#haptics)
    - [notifications](#notifications)
    - [lifecycle events](#lifecycle-events)
  - [creating plugins](#creating-plugins)
    - [plugin structure](#plugin-structure)
    - [scaffolding a plugin](#scaffolding-a-plugin)
    - [ios implementation](#ios-implementation)
    - [android implementation](#android-implementation)
    - [desktop implementation](#desktop-implementation)
  - [plugin registration](#plugin-registration)

<a name="introduction"></a>

## introduction

the native bridge provides bidirectional communication between your javascript code and the native runtime. all device apis are exposed through a unified interface that works identically across ios, android, and desktop platforms.

<a name="the-native-bridge"></a>

## the native bridge

<a name="bridge-architecture"></a>

### bridge architecture

the bridge operates on a request response model. your javascript code sends method calls to the native layer, which executes the operation and returns a result asynchronously.

all communication uses json serialization. complex objects are automatically marshalled across the boundary. promises provide natural async handling on the javascript side.

<a name="calling-native-methods"></a>

### calling native methods

invoke native functionality using the global bridge object:

```javascript
const result = await window.dstrnBridge.call('storage.get', { key: 'username' });
console.log(result.value);
```

the `call` method accepts a method name and optional data payload. it returns a promise that resolves with the native response or rejects on error.

method names use dot notation to namespace functionality. all built in apis use the pattern `category.action`.

<a name="receiving-native-events"></a>

### receiving native events

the native runtime can push events to your javascript code. listen for events using the bridge event emitter:

```javascript
window.dstrnBridge.on('app:background', () => {
  console.log('app moved to background');
});
```

lifecycle events fire automatically when the app state changes. custom plugins can emit their own events using the same mechanism.

<a name="available-apis"></a>

## available apis

<a name="storage"></a>

### storage

persistent key value storage backed by native secure storage apis:

```javascript
await window.dstrnBridge.call('storage.set', { key: 'token', value: 'abc123' });
const result = await window.dstrnBridge.call('storage.get', { key: 'token' });
const keys = await window.dstrnBridge.call('storage.keys');
await window.dstrnBridge.call('storage.remove', { key: 'token' });
await window.dstrnBridge.call('storage.clear');
```

<a name="clipboard"></a>

### clipboard

read and write the system clipboard:

```javascript
await window.dstrnBridge.call('clipboard.write', { text: 'hello world' });
const result = await window.dstrnBridge.call('clipboard.read');
console.log(result.text);
```

<a name="haptics"></a>

### haptics

trigger haptic feedback on supported devices:

```javascript
await window.dstrnBridge.call('haptic.impact', { style: 'medium' });
await window.dstrnBridge.call('haptic.notification', { type: 'success' });
await window.dstrnBridge.call('haptic.selection');
```

<a name="notifications"></a>

### notifications

request permissions and display local notifications:

```javascript
const result = await window.dstrnBridge.call('notification.permission');
if (result.status === 'granted') {
  await window.dstrnBridge.call('notification.send', {
    title: 'update available',
    body: 'a new version is ready',
  });
}
```

<a name="lifecycle-events"></a>

### lifecycle events

the runtime emits events when the application state changes:

```javascript
window.dstrnBridge.on('app:foreground', () => {
  console.log('app resumed');
});

window.dstrnBridge.on('app:background', () => {
  console.log('app paused');
});

window.dstrnBridge.on('app:terminate', () => {
  console.log('app closing');
});
```

<a name="creating-plugins"></a>

## creating plugins

<a name="plugin-structure"></a>

### plugin structure

plugins extend the bridge with custom native functionality. each plugin consists of three platform specific implementations that expose identical javascript apis.

a plugin named `camera` would provide methods like `camera.capture` and `camera.permissions` that work consistently across all platforms despite different underlying implementations.

<a name="scaffolding-a-plugin"></a>

### scaffolding a plugin

generate a plugin template using the cli:

```bash
dstrn make:plugin camera
```

this creates platform specific files in your project:

```
native/plugins/
├── camera.swift         # ios
├── camera.kt            # android
└── camera.rs            # desktop
```

each file contains boilerplate with method registration and example implementations.

<a name="ios-implementation"></a>

### ios implementation

ios plugins extend the bridge class with new methods:

```swift
extension Bridge {
  @objc func cameraCapture(_ data: [String: Any], _ callback: @escaping RCTResponseSenderBlock) {
    let picker = UIImagePickerController()
    picker.sourceType = .camera
    // implementation
    callback([["success": true]])
  }
}
```

<a name="android-implementation"></a>

### android implementation

android plugins add methods to the bridge class:

```kotlin
class Bridge {
  @JavascriptInterface
  fun cameraCapture(data: String): String {
    val intent = Intent(MediaStore.ACTION_IMAGE_CAPTURE)
    // implementation
    return JSONObject().put("success", true).toString()
  }
}
```

<a name="desktop-implementation"></a>

### desktop implementation

desktop plugins use tauri commands:

```rust
#[tauri::command]
fn camera_capture(data: Value) -> Result<Value, String> {
  // implementation
  Ok(json!({"success": true}))
}
```

<a name="plugin-registration"></a>

## plugin registration

after creating platform implementations, rebuild your native runtimes. the framework automatically discovers and registers plugin methods during the build process.

call plugin methods using the same bridge api:

```javascript
const photo = await window.dstrnBridge.call('camera.capture', { 
  quality: 0.8 
});
```

plugins can emit custom events that your javascript code listens for:

```javascript
window.dstrnBridge.on('camera:captured', (data) => {
  console.log('photo saved:', data.path);
});
```
