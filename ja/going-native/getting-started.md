# ネイティブアプリケーション

- [ネイティブアプリケーション](#ネイティブアプリケーション)
  - [はじめに](#はじめに)
  - [サポート対象プラットフォーム](#サポート対象プラットフォーム)
  - [設定](#設定)
    - [基本セットアップ](#基本セットアップ)
    - [アプリケーション識別子](#アプリケーション識別子)
    - [url設定](#url設定)
    - [静的フォールバック](#静的フォールバック)
  - [開発ワークフロー](#開発ワークフロー)
    - [シミュレーション](#シミュレーション)
    - [ビルド](#ビルド)
  - [プロジェクト構成](#プロジェクト構成)

<a name="はじめに"></a>

## はじめに

dframeworkを使用するとWebアプリケーションをios、android、desktopプラットフォーム向けのネイティブアプリとしてパッケージ化できます。ネイティブレイヤーは軽量なwebviewラッパーでありアプリケーションロジックを完全にWebベースに保ちながらjavascriptブリッジを通じてデバイスapiへのアクセスを提供します。

個別のコードベースや複雑なビルド設定を必要とするハイブリッドフレームワークとは異なりdframeworkは単一の信頼できる情報源を維持します。ルート、コントローラー、モデル、ビューはWeb環境とネイティブ環境でまったく同じように動作します。

<a name="サポート対象プラットフォーム"></a>

## サポート対象プラットフォーム

フレームワークは3つのプラットフォーム向けにネイティブランタイムを提供します:

- **ios**（wkwebviewを使用したネイティブswiftアプリケーション）
- **android**（android webviewを使用したネイティブkotlinアプリケーション）
- **desktop**（tauriを使用したクロスプラットフォームアプリケーション（rust + webview））

各ランタイムには自動ネットワーク適応、javascriptポリフィル、統一されたブリッジapiが含まれています。カスタムネイティブプラグインが必要な場合を除きアプリケーションにプラットフォーム固有のコードは必要ありません。

<a name="設定"></a>

## 設定

<a name="基本セットアップ"></a>

### 基本セットアップ

ネイティブ設定はプロジェクトルートの[`config/native.js`](../getting-started/configuration.md#config-native)に配置されます。フレームワークは設定を適切なデフォルト値とマージします。

```javascript
export default {
  name: 'myapp',
  identifier: 'com.example.myapp',
  version: '1.0.0',
  build: 1,
};
```

`name`フィールドを省略した場合フレームワークはフォールバックとして[`Config('app.name')`](../getting-started/configuration.md#app-config)を使用します。

<a name="アプリケーション識別子"></a>

### アプリケーション識別子

> [!IMPORTANT]
> `config/native.js`の`identifier`はリバースドメイン表記（例: `com.example.myapp`）に従う必要があります。この値は配信後に新しいアプリ掲載情報を公開することなく変更することはできません。

```javascript
export default {
  identifier: 'com.example.myapp',
  version: '2.1.0',
  build: 42,
};
```

`version`文字列はユーザー向けです。`build`番号はリリースごとにインクリメントする必要がありアプリストアでバージョンを区別するために使用されます。

<a name="url設定"></a>

### url設定

デフォルトではネイティブアプリは[`Config('app.url')`](../getting-started/configuration.md#app-config)と[`Config('app.port')`](../getting-started/configuration.md#app-config)からコンテンツをロードします。`index`フィールドを設定することでこの動作をオーバーライドできます。

```javascript
export default {
  index: 'https://myapp.com',
};
```

`index`がnullの場合フレームワークはプライマリアプリケーション設定からurlを構築します。これによりapi urlを変更すると手動で同期することなくネイティブビルドが自動的に更新されます。

シミュレーション中フレームワークは環境変数を介して開発サーバーのurlを注入するため本番設定に関係なくlocalhostに対してテストできます。

<a name="静的フォールバック"></a>

### 静的フォールバック

リモートurlにアクセスできない場合のフォールバックとして静的htmlサイトをネイティブアプリケーション内にバンドルできます。これによりオフライン機能を有効にしたりリトライロジックを備えたエラーページを提供したりできます。

```javascript
export default {
  index: 'https://api.myapp.com',
  fallback: 'native/static/index.html',
};
```

静的サイトはバンドルされたdframework cssおよびjavascriptランタイムに完全にアクセスできます。静的ファイルは[`native/static/`](../getting-started/project-structure.md#native-static)ディレクトリに配置してください。フレームワークはビルド中にこのディレクトリをネイティブバンドルにコピーします。

最小限のフォールバックページではリロードボタン付きのネットワークエラーを表示できます:

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

ネイティブランタイムはプライマリurlが失敗したときに自動的にフォールバックのロードを試みます。また`index`をnullに設定し完全な静的実装を提供することで完全なオフラインアプリケーションを構築することもできます。

<a name="開発ワークフロー"></a>

## 開発ワークフロー

<a name="シミュレーション"></a>

### シミュレーション

開発中にネイティブ環境でアプリケーションをテストするには`simulate`コマンドを使用します。サーバーがまだ実行されていない場合フレームワークは自動的にサーバーを起動します。

```bash
dstrn simulate --ios
dstrn simulate --android
dstrn simulate --desktop
```

シミュレーターはローカル開発サーバーに接続します。ブラウザと同様にリフレッシュ後にアプリケーションコードの変更が即座に反映されます。

`r`キーを押してネイティブアプリをリフレッシュします。`q`キーを押して終了します。

<a name="ビルド"></a>

### ビルド

`build`コマンドは配信用の本番バイナリをコンパイルします。ビルドはプラットフォームごとに整理され[`native/builds/`](../getting-started/project-structure.md#native-builds)に出力されます。

```bash
dstrn build --ios
dstrn build --android
dstrn build --desktop
```

本番ビルドでは設定された`index` urlが使用されます。明示的な値を設定していない場合フレームワークは[`config/app.js`](../getting-started/configuration.md#app-config)の本番apiエンドポイントを使用します。

ビルドにはプラットフォーム固有のツールチェーンが必要です。環境セットアップの手順については[ビルドと配信のドキュメント](build-distribution.md)を参照してください。

<a name="プロジェクト構成"></a>

## プロジェクト構成

ネイティブアセットと設定はプロジェクトルートの`native`ディレクトリに配置されます:

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

フレームワークはコンパイル中に`builds`ディレクトリを生成します。バージョン管理からは除外してください。

`icon.png`ファイルは1024x1024ピクセル以上である必要があります。フレームワークはビルドプロセス中に各プラットフォームに必要なすべてのサイズと形式を自動的に生成します。