# build and distribution

- [build and distribution](#build-and-distribution)
  - [introduction](#introduction)
  - [environment setup](#environment-setup)
    - [ios requirements](#ios-requirements)
    - [android requirements](#android-requirements)
    - [desktop requirements](#desktop-requirements)
  - [building applications](#building-applications)
    - [ios builds](#ios-builds)
    - [android builds](#android-builds)
    - [desktop builds](#desktop-builds)
  - [code signing](#code-signing)
    - [ios signing](#ios-signing)
    - [android signing](#android-signing)
  - [distribution](#distribution)
    - [app store submission](#app-store-submission)
    - [google play submission](#google-play-submission)
    - [desktop distribution](#desktop-distribution)
  - [version management](#version-management)

<a name="introduction"></a>

## introduction

production builds compile your application into platform specific binaries ready for distribution. unlike simulation, builds create fully self contained packages that can be installed on physical devices or submitted to app stores.

<a name="environment-setup"></a>

## environment setup

<a name="ios-requirements"></a>

### ios requirements

ios builds require a macos system with xcode installed:

```bash
xcode-select --install
```

verify your installation:

```bash
xcodebuild -version
```

you need an active apple developer account to sign and distribute ios applications. development builds can be installed directly on devices registered in your account. app store distribution requires enrollment in the apple developer program.

<a name="android-requirements"></a>

### android requirements

android builds work on macos, linux, and windows. install the android sdk and platform tools:

```bash
# verify installation
adb --version
```

you need a java runtime. the framework automatically detects android studio's bundled jdk if present.

signing android releases requires a keystore. generate one using the java keytool:

```bash
keytool -genkey -v -keystore release.keystore -alias myapp -keyalg RSA -keysize 2048 -validity 10000
```

store the keystore file outside your project directory. never commit it to version control.

<a name="desktop-requirements"></a>

### desktop requirements

desktop builds require the rust toolchain and tauri cli:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install tauri-cli
```

verify the installation:

```bash
cargo tauri --version
```

desktop builds produce platform specific installers. on macos, the output is a dmg and app bundle. on windows, you get an msi installer and exe. on linux, the framework generates deb and appimage formats.

<a name="building-applications"></a>

## building applications

<a name="ios-builds"></a>

### ios builds

```bash
dstrn build --target ios
```

the build process compiles swift code, generates all required icon sizes, injects your configuration, and archives the application. the output appears in [`native/builds/ios/`](native-getting-started.md#project-structure).

for physical device testing, the build must be signed with a development certificate. xcode handles this automatically if you open the generated archive and select your development team.

for app store submission, build with a distribution certificate and upload using xcode or the transporter app.

<a name="android-builds"></a>

### android builds

```bash
dstrn build --target android
```

this produces an unsigned apk in [`native/builds/android/`](native-getting-started.md#project-structure). you can install unsigned builds directly on physical devices with usb debugging enabled:

```bash
adb install native/builds/android/app-release.apk
```

for play store submission, sign the apk with your release keystore using the jarsigner tool, then optimize it with zipalign.

<a name="desktop-builds"></a>

### desktop builds

```bash
dstrn build --target desktop
```

the tauri build process compiles rust code and generates installers for your current platform. builds take several minutes on first run as dependencies are compiled.

output location depends on your operating system:
- macos: dmg and app bundle in [`native/builds/desktop/`](native-getting-started.md#project-structure)
- windows: msi installer and standalone exe
- linux: deb package and appimage

desktop applications do not require code signing for local distribution but should be signed for wider distribution to avoid security warnings.

<a name="code-signing"></a>

## code signing

<a name="ios-signing"></a>

### ios signing

ios requires all applications to be signed. development signing happens automatically when you select a development team in xcode. distribution signing requires:

1. apple developer account with active membership
2. distribution certificate installed in keychain
3. provisioning profile for your app identifier

open the generated xcarchive in xcode, select your distribution certificate, and export for app store submission or ad hoc distribution.

<a name="android-signing"></a>

### android signing

sign the release apk using your keystore:

```bash
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 -keystore release.keystore app-release.apk myapp
```

optimize the signed apk:

```bash
zipalign -v 4 app-release.apk app-release-aligned.apk
```

store your keystore password securely. losing the keystore means you cannot update your application on the play store.

<a name="distribution"></a>

## distribution

<a name="app-store-submission"></a>

### app store submission

prepare your app store listing with screenshots, description, and metadata. upload the signed ipa using xcode or apple's transporter application.

the review process typically takes 24 to 48 hours. ensure your configured url points to a production server that apple's reviewers can access.

<a name="google-play-submission"></a>

### google play submission

create a new application in the google play console. upload the signed apk and complete the store listing with screenshots and description.

google play supports staged rollouts. you can release to a percentage of users first and gradually increase availability after monitoring for issues.

<a name="desktop-distribution"></a>

### desktop distribution

desktop applications can be distributed directly to users as downloadable installers. host the files on your website or use a service like github releases.

for wider distribution, consider:
- macos: notarization with apple and distribution via the mac app store
- windows: code signing certificate to avoid smartscreen warnings
- linux: submit to package repositories like snapcraft or flathub

<a name="version-management"></a>

## version management

increment the version number and build number in [`config/native.js`](native-getting-started.md#application-identity) before each release:

```javascript
export default {
  version: '1.2.0',
  build: 15,
};
```

the version string is user facing and follows semantic versioning. the build number must increment with every submission to app stores, even if the version string stays the same.

app stores reject submissions with duplicate build numbers. automate version bumps in your release scripts to avoid this issue.
