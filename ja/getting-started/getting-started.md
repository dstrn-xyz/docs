# スタートガイド

- [dframeworkの紹介](#dframeworkの紹介)

  - [なぜdframeworkなのか？](#なぜdframeworkなのか？)

- [アプリケーションの作成](#アプリケーションの作成)

  - [フレームワークのインストール](#フレームワークのインストール)
  - [プロジェクトの初期化](#プロジェクトの初期化)
  - [ランタイムの起動](#ランタイムの起動)

- [初期設定](#初期設定)

  - [環境設定](#環境設定)

<a name="dframeworkの紹介"></a>

## dframeworkの紹介

dframeworkは妥協のないアプリケーション開発アプローチを具現化したフレームワークです。柔軟性よりもアーキテクチャの一貫性を優先し、リクエストライフサイクルを完全に制御します。競合するパラダイムや交換可能な層は存在しません。フレームワークそのものがアプリケーションです。

<a name="なぜdframeworkなのか？"></a>

### なぜdframeworkなのか？

現代の開発は断片化したツール、多層の抽象化、交換可能なアーキテクチャへの依存を強めています。これが設定疲れやアーキテクチャの不整合を生み出しています。dframeworkは厳格で統合された環境を強制することで、正反対のアプローチをとります。

<a name="アプリケーションの作成"></a>

## アプリケーションの作成

<a name="フレームワークのインストール"></a>

### フレームワークのインストール

開始するにはグローバルフレームワークバイナリをインストールする必要があります。npm経由で`@dstrn/framework`パッケージをグローバルにインストールします。これによりアプリケーションの初期化や管理に必要なコマンドラインツールが提供されます。

```bash
npm install -g @dstrn/framework
```

<a name="プロジェクトの初期化"></a>

### プロジェクトの初期化

バイナリがシステムで利用可能になったら、新しいアプリケーションを生成できます。initコマンドを使用しアプリケーション名を指定します。これによりフレームワークが必要とする正確なディレクトリ構造が構築されます。

```bash
dstrn init myapp
cd myapp
```

<a name="ランタイムの起動"></a>

### ランタイムの起動

dframeworkはディレクトリ構造と特定の起動機構を強制します。アプリケーションは提供されたコマンドラインツールを介して起動されます。serveコマンドはアプリケーションランタイムを初期化し、ビューをコンパイルして内部サーバーを起動します。

```bash
dstrn serve
```

<a name="初期設定"></a>

## 初期設定

<a name="環境設定"></a>

### 環境設定

内部的にはフレームワークは中央のアプリケーションインスタンスを利用します。このインスタンスはルーターからビューエンジン、データベース接続に至るまですべてを制御します。環境を自動的に検出し、それに応じてリクエストパイプラインを構成します。

アプリケーションが起動するとデータベース接続を登録し、ロガーを初期化し、リクエストハンドラーを準備します。起動プロセス中にフレームワークはネイティブソケットルーターおよびバックグラウンドキューワーカーも解決します。

> [!NOTE]
> データベース接続、ロガー、ソケットルーター、キューワーカーを手動でインスタンス化することは決してありません。これらはフレームワークランタイムによってのみ解決および管理されます。

<br />

プロジェクトルートに`.env`ファイルを作成します。すべての値は起動時に解析されます。

`config`ディレクトリには`Env.value()`を使用して環境変数を読み取るモジュール化された設定ファイルが配置されます。
以下は基本的な設定の例です:

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
  connectionLimit: Env.value('DB_CONNECTION_LIMIT', 10),
  maxIdle: Env.value('DB_MAX_IDLE', 5),
  idleTimeout: Env.value('DB_IDLE_TIMEOUT', 60000),
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
      root: Env.value('STORAGE_ROOT', './storage/local'),
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

設定の完全な詳細については専用の[設定ドキュメント](configuration.md)を参照してください。
