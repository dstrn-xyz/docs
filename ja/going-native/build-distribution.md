# ビルドと配信

- [ビルドと配信](#ビルドと配信)
  - [はじめに](#はじめに)
  - [環境のセットアップ](#環境のセットアップ)
    - [ios要件](#ios要件)
    - [android要件](#android要件)
    - [desktop要件](#desktop要件)
  - [アプリケーションのビルド](#アプリケーションのビルド)
    - [iosビルド](#iosビルド)
    - [androidビルド](#androidビルド)
    - [desktopビルド](#desktopビルド)
  - [コード署名](#コード署名)
    - [ios署名](#ios署名)
    - [android署名](#android署名)
  - [配信](#配信)
    - [app storeへの提出](#app-storeへの提出)
    - [google playへの提出](#google-playへの提出)
    - [desktop配信](#desktop配信)
  - [バージョン管理](#バージョン管理)

<a name="はじめに"></a>

## はじめに

本番ビルドはアプリケーションを配信可能なプラットフォーム固有のバイナリにコンパイルします。シミュレーションとは異なりビルドでは実機デバイスにインストールしたりアプリストアに提出したりできる完全に自己完結したパッケージが作成されます。

<a name="環境のセットアップ"></a>

## 環境のセットアップ

<a name="ios要件"></a>

### ios要件

iosビルドにはxcodeがインストールされたmacosシステムが必要です:

```bash
xcode-select --install
```

インストールを確認します:

```bash
xcodebuild -version
```

iosアプリケーションに署名して配信するにはアクティブなapple developerアカウントが必要です。開発ビルドはアカウントに登録されたデバイスに直接インストールできます。app storeへの配信にはapple developer programへの登録が必要です。

<a name="android要件"></a>

### android要件

androidビルドはmacos、linux、windowsで動作します。android sdkとプラットフォームツールをインストールします:

```bash
# verify installation
adb --version
```

javaランタイムが必要です。android studioのバンドルjdkが存在する場合フレームワークはそれを自動検出します。

androidリリースの署名にはキーストアが必要です。java keytoolを使用して生成します:

```bash
keytool -genkey -v -keystore release.keystore -alias myapp -keyalg RSA -keysize 2048 -validity 10000
```

キーストアファイルはプロジェクトディレクトリの外に保管してください。バージョン管理には絶対にコミットしないでください。

<a name="desktop要件"></a>

### desktop要件

desktopビルドにはrustツールチェーンとtauri cliが必要です:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install tauri-cli
```

インストールを確認します:

```bash
cargo tauri --version
```

desktopビルドはプラットフォーム固有のインストーラーを生成します。macosではdmgとappバンドルが出力されます。windowsではmsiインストーラーとexeが出力されます。linuxではdebおよびappimage形式が生成されます。

<a name="アプリケーションのビルド"></a>

## アプリケーションのビルド

<a name="iosビルド"></a>

### iosビルド

```bash
dstrn build --ios
```

ビルドプロセスはswiftコードをコンパイルし必要なすべてのアイコンサイズを生成し設定を注入してアプリケーションをアーカイブします。出力は[`native/builds/ios/`](getting-started.md#プロジェクト構成)に保存されます。

実機デバイスでのテストではビルドに開発証明書で署名する必要があります。生成されたアーカイブを開いて開発チームを選択するとxcodeがこれを自動的に処理します。

app storeへの提出には配信証明書でビルドしxcodeまたはtransporterアプリを使用してアップロードします。

<a name="androidビルド"></a>

### androidビルド

```bash
dstrn build --android
```

これにより[`native/builds/android/`](getting-started.md#プロジェクト構成)に未署名のapkが生成されます。usbデバッグを有効にした実機デバイスに未署名ビルドを直接インストールできます:

```bash
adb install native/builds/android/app-release.apk
```

play storeへの提出にはjarsignerツールを使用してリリース用キーストアでapkに署名しzipalignで最適化します。

<a name="desktopビルド"></a>

### desktopビルド

```bash
dstrn build --desktop
```

tauriのビルドプロセスはrustコードをコンパイルし現在のプラットフォーム向けのインストーラーを生成します。依存関係がコンパイルされるため初回実行時には数分かかります。

出力先はオペレーティングシステムによって異なります:
- macos: [`native/builds/desktop/`](getting-started.md#プロジェクト構成)内のdmgおよびappバンドル
- windows: msiインストーラーおよびスタンドアロンexe
- linux: debパッケージおよびappimage

デスクトップアプリケーションはローカル配信のためにコード署名を必要としませんがセキュリティ警告を避けるために広範な配信時には署名することをお勧めします。

<a name="コード署名"></a>

## コード署名

<a name="ios署名"></a>

### ios署名

iosではすべてのアプリケーションに署名が必要です。xcodeで開発チームを選択すると開発署名が自動的に行われます。配信署名には以下が必要です:

1. 有効なメンバーシップを持つapple developerアカウント
2. キーチェーンにインストールされた配信証明書
3. アプリ識別子に対応するプロビジョニングプロファイル

生成されたxcarchiveをxcodeで開き配信証明書を選択してapp storeへの提出用またはアドホック配信用にエクスポートします。

<a name="android署名"></a>

### android署名

キーストアを使用してリリースapkに署名します:

```bash
jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 -keystore release.keystore app-release.apk myapp
```

署名済みapkを最適化します:

```bash
zipalign -v 4 app-release.apk app-release-aligned.apk
```

> [!CAUTION]
> リリース用キーストアとパスワードはプロジェクトリポジトリの外部に安全に保管してください。キーストアを紛失すると既存のgoogle playアプリケーションを更新できなくなります。

<a name="配信"></a>

## 配信

<a name="app-storeへの提出"></a>

### app storeへの提出

スクリーンショット、説明、メタデータを準備してapp storeの掲載情報を設定します。xcodeまたはappleのtransporterアプリケーションを使用して署名済みipaをアップロードします。

審査プロセスには通常24から48時間かかります。設定したurlがappleの審査担当者がアクセスできる本番サーバーを指していることを確認してください。

<a name="google-playへの提出"></a>

### google playへの提出

google play consoleで新しいアプリケーションを作成します。署名済みapkをアップロードしスクリーンショットと説明を入力してストア掲載情報を完成させます。

google playは段階的ロールアウトをサポートしています。最初にユーザーの一部にリリースし問題を監視した後に徐々に公開範囲を広げることができます。

<a name="desktop配信"></a>

### desktop配信

デスクトップアプリケーションはダウンロード可能なインストーラーとしてユーザーに直接配信できます。Webサイトでファイルをホストするかgithub releasesなどのサービスを使用します。

より広範な配信を行う場合は以下を検討してください:
- macos: appleによる公証およびmac app store経由の配信
- windows: smartscreen警告を回避するためのコード署名証明書
- linux: snapcraftやflathubなどのパッケージリポジトリへの提出

<a name="バージョン管理"></a>

## バージョン管理

各リリース前に[`config/native.js`](getting-started.md#アプリケーション識別子)のバージョン番号とビルド番号をインクリメントします:

```javascript
export default {
  version: '1.2.0',
  build: 15,
};
```

version文字列はユーザー向けでありセマンティックバージョニングに従います。

> [!IMPORTANT]
> ユーザー向けの`version`文字列が変更されていない場合でもapp storeへの提出ごとに`build`整数をインクリメントする必要があります。重複したビルド番号を持つ提出はapp storeによって拒否されます。