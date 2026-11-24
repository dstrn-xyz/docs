# 設定

- [はじめに](#はじめに)
- [環境ファイル](#環境ファイル)
- [設定ファイル](#設定ファイル)
  - [config/app.js](#config/app.js)
  - [config/auth.js](#config/auth.js)
  - [config/database.js](#config/database.js)
  - [config/session.js](#config/session.js)
  - [config/storage.js](#config/storage.js)
  - [config/socket.js](#config/socket.js)
  - [config/native.js](#config/native.js)
  - [config/view.js](#config/view.js)
  - [config/tracking.js](#config/tracking.js)
- [設定値へのアクセス](#設定値へのアクセス)
- [環境変数へのアクセス](#環境変数へのアクセス)
- [設定のホットリロード](#設定のホットリロード)

<a name="はじめに"></a>

## はじめに

すべてのアプリケーション設定はプロジェクトルートの`.env`ファイルと`config`ディレクトリの2か所に存在します。`.env`ファイルにはシークレットやデータベース認証情報などの環境固有の値を保持します。`config`ディレクトリには環境変数を読み込んでアプリケーション全体へ型変換された値を公開する構造化されたJavaScriptファイルを配置します。フレームワークは起動時に両方を自動的にロードします。

<a name="環境ファイル"></a>

## 環境ファイル

`.env`ファイルはプロジェクトのルートに配置され生の環境変数の割り当てを保持します。提供されている`.env.example`を`.env`にコピーして値を設定します。`.env`ファイルはバージョン管理にコミットしません。

```bash
APP_NAME=dframework
APP_ENV=local
APP_DEBUG=true
APP_URL=http://localhost
APP_HOST=null
APP_PORT=8250
APP_LOCALE=en
APP_KEY=
APP_TRANSPORT=null
HASH_FIELDS=password

MAX_BODY_SIZE=1
MAX_UPLOAD_SIZE=50
TRUST_PROXY=false

SHIELD_ENABLED=false
SHIELD_DIFFICULTY=4
SHIELD_THRESHOLD=70

MAX_COMPILED_CACHE_SIZE=500
MAX_TEMPLATE_SOURCE_CACHE_SIZE=500

STATIC_FILE_CACHE_MAX_FILES=100
STATIC_FILE_CACHE_MAX_SIZE=1048576
STATIC_FILE_CACHE_MAX_TOTAL_BYTES=33554432

OPTIMIZE_CSS=true

TLS_CERT=null
TLS_KEY=null
TLS_CA=null
TLS_REDIRECT=true

QUEUE_MAX_WORKERS=4
QUEUE_TIMEOUT=60000
QUEUE_SHUTDOWN_TIMEOUT=30000

DB_DRIVER=mysql
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASS=
DB_NAME=dframework
DB_CONNECTION_LIMIT=10
DB_MAX_IDLE=5
DB_IDLE_TIMEOUT=60000
DB_QUERY_CACHE_MAX_ENTRIES=200
DB_QUERY_CACHE_MAX_ROWS=1000

SESSION_DRIVER=memory
SESSION_COOKIE=sid
SESSION_LIFETIME=120
SESSION_DOMAIN=null
SESSION_PATH=/
SESSION_SAME_SITE=Lax
SESSION_HTTP_ONLY=true
SESSION_SECURE=null
SESSION_MAX_SESSIONS=10000

STORAGE_ROOT=./storage
STORAGE_PUBLIC=./storage/public
```

フレームワークは起動時にこのファイルを解析します。引用符のない整数文字列（`8250`や`-5`など）は安全な整数にキャストされます。引用符のない`true`、`false`、`null`の値は真偽値およびnullプリミティブにキャストされます。引用符で囲まれた値（単一引用符または二重引用符で囲まれた値）は厳密に文字列として保持され、数値のみのパスワード、ハッシュ、トークン、または先頭にゼロを含む文字列が型変換されるのを防ぎます。これらの値は設定ファイル全体で`Env.value()`を介して即座に利用可能です。

> [!IMPORTANT]
> `local`環境でアプリケーションの実行中に`.env`ファイルを変更した場合、フレームワークは新しい値を反映するためにサーバー全体を再起動します。

<a name="設定ファイル"></a>

## 設定ファイル

`config`ディレクトリ内の各ファイルはデフォルトのプレーンオブジェクトをエクスポートし、`Env.value()`を使用して環境変数から値を取得します。ファイル名はアプリケーション全体でドット記法ルックアップに使用される設定名前空間になります。

<a name="config/app.js"></a>

### config/app.js

メインの設定ファイルであり、アプリケーションのコアランタイム動作を定義します。

```javascript
// config/app.js
import { Env } from 'dframework';

export default {
  name: Env.value('APP_NAME', 'dframework'),
  env: Env.value('APP_ENV', 'production'),
  debug: Env.value('APP_DEBUG', false),
  url: Env.value('APP_URL', 'http://localhost'),
  host: Env.value('APP_HOST', null),
  port: Env.value('APP_PORT', 8250),
  locale: Env.value('APP_LOCALE', 'en'),
  basePath: Env.value('APP_BASE_PATH', null),
  langPath: Env.value('APP_LANG_PATH', null),
  key: Env.value('APP_KEY'),
  hashFields: (Env.value('HASH_FIELDS') || 'password').split(',').map(s => s.trim()).filter(Boolean),

  maxBodySize: Env.value('MAX_BODY_SIZE', 1),
  maxUploadSize: Env.value('MAX_UPLOAD_SIZE', 50),
  trustProxy: Env.value('TRUST_PROXY', false),
  transport: Env.value('APP_TRANSPORT', null),

  shield: Env.value('SHIELD_ENABLED', false),
  shield_difficulty: Env.value('SHIELD_DIFFICULTY', 4),
  shield_threshold: Env.value('SHIELD_THRESHOLD', 70),

  maxCompiledCacheSize: Env.value('MAX_COMPILED_CACHE_SIZE', 500),
  maxTemplateSourceCacheSize: Env.value('MAX_TEMPLATE_SOURCE_CACHE_SIZE', 500),

  staticFileCache: {
    maxFiles: Env.value('STATIC_FILE_CACHE_MAX_FILES', 100),
    maxSize: Env.value('STATIC_FILE_CACHE_MAX_SIZE', 1024 * 1024),
    maxTotalBytes: Env.value('STATIC_FILE_CACHE_MAX_TOTAL_BYTES', 32 * 1024 * 1024),
  },

  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
    safelist: [],
  },

  tls: {
    cert: Env.value('TLS_CERT', null),
    key: Env.value('TLS_KEY', null),
    ca: Env.value('TLS_CA', null),
    redirect: Env.value('TLS_REDIRECT', true),
  },

  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
    timeout: Env.value('QUEUE_TIMEOUT', 60000),
    bootstrapTimeout: Env.value('QUEUE_BOOTSTRAP_TIMEOUT', 30000),
    shutdownTimeout: Env.value('QUEUE_SHUTDOWN_TIMEOUT', 30000),
  },
};
```

<a name="config-tuning"></a>

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `name` | `APP_NAME` | 文字列 | `'dframework'` | バナーやログに表示されるアプリケーション名 |
| `env` | `APP_ENV` | `'production' \| 'local'` | `'production'` | ランタイム環境モード |
| `debug` | `APP_DEBUG` | 真偽値（`true \| false`） | `false` | localモードで詳細なエラー診断トレースを有効化 |
| `url` | `APP_URL` | 文字列（有効なURL） | `'http://localhost'` | リンク、リダイレクト、CLIバナー用のベースURL |
| `host` | `APP_HOST` | 文字列 | `null`（例: `'0.0.0.0'`、`'127.0.0.1'`、`null`） | HTTPサーバーがバインドするホストアドレス（nullは全インターフェースにバインド） |
| `port` | `APP_PORT` | 整数（1から65535） | `8250` | HTTPサーバーのTCPポート番号 |
| `locale` | `APP_LOCALE` | 文字列（ISO言語コード 例: `'en'`、`'ja'`） | `'en'` | ビューローカライゼーションのデフォルト言語 |
| `basePath` | `APP_BASE_PATH` | 文字列（ディレクトリパス）\| null | `process.cwd()` | コントローラー、ジョブ、ミドルウェアのカスタムルートディレクトリ |
| `langPath` | `APP_LANG_PATH` | 文字列（ディレクトリパス）\| null | `null`（ベースパス配下の`lang`を解決） | ローカライゼーション辞書を含むカスタムディレクトリ |
| `key` | `APP_KEY` | 文字列（32文字以上） | なし（本番環境では必須） | セッション署名およびハッシュ化に使用されるアプリケーションシークレット |
| `hashFields` | `HASH_FIELDS` | 文字列配列（環境変数内ではカンマ区切り） | `['password']` | データベース操作中に自動的にハッシュ化されるリクエストパラメータ |
| `maxBodySize` | `MAX_BODY_SIZE` | 整数（メガバイト） | `1` | 受信リクエストペイロードボディの最大サイズ（MB単位） |
| `maxUploadSize` | `MAX_UPLOAD_SIZE` | 整数（メガバイト） | `50` | マルチパートファイルアップロードペイロードの最大サイズ（MB単位） |
| `trustProxy` | `TRUST_PROXY` | 真偽値（`true \| false`） | `false` | x-forwarded-forおよびx-forwarded-protoプロキシヘッダーを信頼 |
| `transport` | `APP_TRANSPORT` | `null \| 'http'` | `null` | HTTPサーバートランスポートエンジン（nullはネイティブdfwsを自動検出しnode-httpへフォールバック） |
| `shield` | `SHIELD_ENABLED` | 真偽値（`true \| false`） | `false` | 対ボットProof of Workチャレンジ保護を有効化 |
| `shield_difficulty` | `SHIELD_DIFFICULTY` | 整数（正の整数） | `4` | Proof of Workパズルの先頭16進ゼロ難易度 |
| `shield_threshold` | `SHIELD_THRESHOLD` | 整数（0から100） | `70` | Proof of Work合格スコア |
| `maxCompiledCacheSize` | `MAX_COMPILED_CACHE_SIZE` | 整数（正の整数） | `500` | メモリにキャッシュされるコンパイル済みビュー関数の最大数 |
| `maxTemplateSourceCacheSize` | `MAX_TEMPLATE_SOURCE_CACHE_SIZE` | 整数（正の整数） | `500` | メモリにキャッシュされる生のテンプレートファイルの最大数 |
| `staticFileCache.maxFiles` | `STATIC_FILE_CACHE_MAX_FILES` | 整数（正の整数） | `100` | メモリに保持される静的公開アセットの最大数 |
| `staticFileCache.maxSize` | `STATIC_FILE_CACHE_MAX_SIZE` | 整数（バイト） | `1048576`（1 MB） | 静的キャッシュの対象となる個別の最大ファイルサイズ |
| `staticFileCache.maxTotalBytes` | `STATIC_FILE_CACHE_MAX_TOTAL_BYTES` | 整数（バイト） | `33554432`（32 MB） | 静的ファイルメモリキャッシュの総割り当て上限 |
| `css.optimize` | `OPTIMIZE_CSS` | 真偽値（`true \| false`） | `true` | 本番ビルド時にCSSのパージと最小化を有効化 |
| `css.safelist` | なし | 文字列配列 | `[]` | パージによる削除から保護されるCSSクラス名 |
| `tls.cert` | `TLS_CERT` | 文字列（ファイルパス）\| null | `null` | TLS SSL証明書ファイルへのパス |
| `tls.key` | `TLS_KEY` | 文字列（ファイルパス）\| null | `null` | TLS秘密鍵ファイルへのパス |
| `tls.ca` | `TLS_CA` | 文字列（ファイルパス）\| null | `null` | 中間認証局バンドルファイルへのパス |
| `tls.redirect` | `TLS_REDIRECT` | 真偽値（`true \| false`） | `true` | 平文HTTPトラフィックをHTTPSへ自動リダイレクト |
| `queue.maxWorkers` | `QUEUE_MAX_WORKERS` | 整数（正の整数） | `4` | バックグラウンドキューワーカープロセスの最大同時実行数 |
| `queue.timeout` | `QUEUE_TIMEOUT` | 整数（ミリ秒） | `60000` | キューに入れられたジョブが強制終了されるまでの実行タイムアウト |
| `queue.bootstrapTimeout` | `QUEUE_BOOTSTRAP_TIMEOUT` | 整数（ミリ秒） | `30000` | 新規キューワーカーの起動完了までのタイムアウト（超過時は強制終了） |
| `queue.shutdownTimeout` | `QUEUE_SHUTDOWN_TIMEOUT` | 整数（ミリ秒） | `30000` | キューワーカーのグレースフルシャットダウン待機タイムアウト |

<a name="config/auth.js"></a>

### config/auth.js

認証設定ではフレームワークが認証済みユーザーを解決するために使用するモデルとガードを登録します。

#### 単一モデル認証

アプリケーション全体で単一のモデルが認証を処理する標準的な構成の場合:

```javascript
// config/auth.js
export default {
  model: 'User',
};
```

#### マルチガード認証

アプリケーションで複数のユーザーエンティティ（一般ユーザーと管理者など）を使用する場合は明示的なガードを設定します:

```javascript
// config/auth.js
export default {
  default: 'web',
  guards: {
    web: {
      model: 'User',
      sessionKey: 'userId',
    },
    admin: {
      model: 'Admin',
      sessionKey: 'adminId',
    },
  },
};
```

#### HTTP Basic認証

保護されたルートでBasic認証ヘッダーを有効にするには、`basic`キーの下に認証情報を定義します:

```javascript
// config/auth.js
export default {
  model: 'User',
  basic: {
    username: 'admin',
    password: 'password',
    realm: 'admin area',
  },
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `model` | なし | 文字列（`models/`配下のモデルクラス名） | `'User'` | 単一ガード認証で使用されるプライマリモデルクラス |
| `default` | なし | 文字列（ガード名） | `'web'` | 認証済みユーザーを解決する際に使用されるデフォルトガード |
| `guards` | なし | ガード定義オブジェクト | `{ web: { model: 'User', sessionKey: 'userId' } }` | ガード名とモデルおよびセッションキーのマッピング |
| `guards.<name>.model` | なし | 文字列（`models/`配下のモデルクラス名） | `'User'` | このガードで解決されるモデルクラス |
| `guards.<name>.sessionKey` | なし | 文字列（セッションキー名） | `'userId'`（web/defaultの場合）または`<name>Id` | 認証済みユーザーIDを格納するセッションキー |
| `basic.username` | なし | 文字列 | なし | HTTP Basic認証に必要なユーザー名 |
| `basic.password` | なし | 文字列 | なし | HTTP Basic認証に必要なパスワード |
| `basic.realm` | なし | 文字列 | `'Restricted'` | 401 www-authenticateヘッダーで返されるレルムプロンプト文字列 |

<a name="config/database.js"></a>

### config/database.js

データベース設定では方言ドライバー、接続認証情報、プール制限、クエリキャッシュパラメータを定義します:

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
  queryCache: {
    maxEntries: Env.value('DB_QUERY_CACHE_MAX_ENTRIES', 200),
    maxRows: Env.value('DB_QUERY_CACHE_MAX_ROWS', 1000),
  },
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `driver` | `DB_DRIVER` | `'mysql' \| 'pgsql' \| 'sqlite'` | `'mysql'` | データベースエンジンの方言ドライバー |
| `host` | `DB_HOST` | 文字列（ホスト名またはIP） | `'localhost'` | データベースサーバーホストアドレス（SQLiteでは未使用） |
| `port` | `DB_PORT` | 整数（1から65535） | `3306`（mysql）または`5432`（pgsql） | データベースサーバーポート番号 |
| `user` | `DB_USER` | 文字列 | `'root'`（mysql）または`'postgres'`（pgsql） | データベースユーザーアカウント |
| `pass` | `DB_PASS` | 文字列 | `''` | データベースユーザーパスワード |
| `name` | `DB_NAME` | 文字列 | アプリケーションデータベース名、またはSQLiteでは`':memory:'` | データベース名またはSQLiteデータベースファイルパス |
| `connectionLimit` | `DB_CONNECTION_LIMIT` | 整数（正の整数） | 本番環境では`20` / local環境では`10` | プール内の最大同時データベース接続数 |
| `maxIdle` | `DB_MAX_IDLE` | 整数（正の整数） | `5` | プール内に保持される最大アイドル接続数 |
| `idleTimeout` | `DB_IDLE_TIMEOUT` | 整数（ミリ秒） | `60000` | アイドル接続を切断するまでの時間 |
| `queryCache.maxEntries` | `DB_QUERY_CACHE_MAX_ENTRIES` | 整数（正の整数） | `200` | LRUクエリキャッシュ内の最大キャッシュクエリプラン数 |
| `queryCache.maxRows` | `DB_QUERY_CACHE_MAX_ROWS` | 整数（正の整数） | `1000` | LRUクエリキャッシュの対象となる結果あたりの最大行数 |

<a name="config/session.js"></a>

### config/session.js

セッション設定ではユーザーセッションのドライバー、クッキー名、有効期間、クッキーパラメータを定義します:

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
  maxSessions: Env.value('SESSION_MAX_SESSIONS', 10000),
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `driver` | `SESSION_DRIVER` | `'memory' \| 'database' \| 'stealth'` | `'memory'` | セッションストレージ機構のバックエンド |
| `cookie` | `SESSION_COOKIE` | 文字列（有効なクッキー名） | `'sid'` | クライアントに送信されるセッションIDクッキー名 |
| `lifetime` | `SESSION_LIFETIME` | 整数（分単位） | `120` | 有効期限までのセッション有効期間（分単位） |
| `domain` | `SESSION_DOMAIN` | 文字列 \| null | `null` | セッションクッキーのドメインスコープ |
| `path` | `SESSION_PATH` | 文字列 | `'/'` | セッションクッキーのパススコープ |
| `same_site` | `SESSION_SAME_SITE` | `'Lax' \| 'Strict' \| 'None'` | `'Lax'` | SameSiteクッキーポリシー |
| `http_only` | `SESSION_HTTP_ONLY` | 真偽値（`true \| false`） | `true` | JavaScriptによるセッションクッキーへのアクセスを防止 |
| `secure` | `SESSION_SECURE` | 真偽値 \| null（`true \| false \| null`） | `null` | セッションクッキーにHTTPSを要求（nullはリクエストから自動検出） |
| `maxSessions` | `SESSION_MAX_SESSIONS` | 整数（正の整数） | `10000` | メモリドライバーによって保持される最大アクティブセッション数 |

<a name="config/storage.js"></a>

### config/storage.js

ストレージ設定ではStorageファサードを通じて利用可能なディスクを定義します:

```javascript
// config/storage.js
import { Env } from 'dframework';

export default {
  default: 'public',
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

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `default` | なし | 文字列（ディスク名） | `'public'` | disk()で省略された場合に使用されるデフォルトストレージディスク名 |
| `disks` | なし | ディスク定義オブジェクト | `{ local, public, secure }` | ディスク名と設定オブジェクトのマッピング |
| `disks.<name>.root` | `STORAGE_ROOT`、`STORAGE_PUBLIC`など | 文字列（ディレクトリパス） | なし（各ディスクで必須） | ディスクルートのローカルファイルシステムディレクトリパス |
| `disks.<name>.encryptionKey` | `APP_KEY`またはカスタムシークレット | 文字列（32文字以上）\| null | `null` | AES 256 GCM自動暗号化および復号化用のキー |

<a name="config/socket.js"></a>

### config/socket.js

ソケット設定ではリアルタイムWebSocketの動作と接続制限を制御します:

```javascript
// config/socket.js
import { Env } from 'dframework';

export default {
  maxPayload: Env.value('SOCKET_MAX_PAYLOAD', 1),
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `maxPayload` | `SOCKET_MAX_PAYLOAD` | 数値（メガバイト） | `1` | WebSocketフレームの最大ペイロードサイズ（MB単位） |

<a name="config/native.js"></a>

### config/native.js

ネイティブ設定ではアプリケーションをネイティブモバイルおよびデスクトップターゲット向けにビルドするためのすべての項目を定義します。このファイルは`config/app.js`に加えてCLIカーネルによってロードされます。

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

| キー | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- |
| `name` | 文字列 \| null | `null`（`app.name`へフォールバック） | ホーム画面の表示名およびアプリケーションウィンドウタイトル |
| `index` | 文字列（URLパス）\| null | `null` | ネイティブコンテナによって読み込まれる初期Webルート |
| `fallback` | 文字列（ファイルパス） | `'native/static/index.html'` | リモートサーバーに接続できない場合のフォールバック静的HTMLファイル |
| `identifier` | 文字列（逆ドメイン形式） | `'com.dframework.app'` | AppleおよびAndroidパッケージ用の一意なバンドル識別子 |
| `version` | 文字列（セマンティックバージョン） | `'1.0.0'` | ユーザー向けアプリケーションバージョン文字列 |
| `build` | 整数（正の整数） | `1` | インクリメンタルビルド番号（Androidバージョンコード） |
| `icon.source` | 文字列（ファイルパス） | `'native/assets/icon.png'` | ソースアイコン画像パス（1024x1024 PNG） |
| `icon.background` | 文字列（16進カラーコード） | `'#ffffff'` | Android向けアダプティブアイコンの背景色 |
| `icon.padding` | 数値（0から1の小数） | `0` | アイコンのインセットパディング比率 |
| `splash.enabled` | 真偽値（`true \| false`） | `true` | ネイティブ起動スプラッシュ画面を有効化 |
| `splash.background` | 文字列（16進カラーコード） | `'#0a0a0a'` | スプラッシュ画面の背景色 |
| `splash.icon` | 文字列（ファイルパス） | `'native/assets/icon.png'` | スプラッシュ画面の中央アイコン画像パス |
| `splash.iconSize` | 数値（0から1の小数） | `0.3` | 画面領域に対するスプラッシュアイコンの縮尺比 |
| `splash.fadeOut` | 整数（ミリ秒） | `300` | スプラッシュ画面消去時のトランジション時間（ミリ秒） |
| `theme.statusBar` | `'light' \| 'dark'` | `'dark'` | ネイティブステータスバーアイコンの外観 |
| `theme.primary` | 文字列（16進カラーコード） | `'#d3ac5f'` | プライマリブランドアクセントカラー |
| `theme.background` | 文字列（16進カラーコード） | `'#0a0a0a'` | WebViewおよびウィンドウコンテナの背景色 |
| `theme.navigationBar` | 文字列（16進カラーコード） | `'#0a0a0a'` | Android下部システムナビゲーションバーの色 |
| `theme.safeArea` | 真偽値（`true \| false`） | `true` | WebViewコンテンツがデバイスのセーフエリアインセットを尊重 |
| `ios.orientation` | `'all' \| 'portrait' \| 'landscape'` | `'all'` | iOSで許可されるデバイス画面の向き |
| `ios.statusBarHidden` | 真偽値（`true \| false`） | `false` | iOSステータスバーを非表示にする |
| `android.orientation` | `'unspecified' \| 'portrait' \| 'landscape'` | `'unspecified'` | Androidで許可される画面の向き |
| `android.minSdk` | 整数（SDK APIレベル） | `24` | サポートされる最小Android SDKバージョン |
| `android.targetSdk` | 整数（SDK APIレベル） | `34` | ターゲットAndroid SDKバージョン |
| `android.permissions` | 文字列配列 | `['INTERNET']` | Androidマニフェストで宣言されるパーミッション |
| `desktop.width` | 整数（ピクセル） | `1280` | デスクトップアプリケーションのデフォルトウィンドウ幅 |
| `desktop.height` | 整数（ピクセル） | `720` | デスクトップアプリケーションのデフォルトウィンドウ高さ |
| `desktop.minWidth` | 整数（ピクセル）\| undefined | `800` | 許可される最小デスクトップウィンドウ幅 |
| `desktop.minHeight` | 整数（ピクセル）\| undefined | `600` | 許可される最小デスクトップウィンドウ高さ |
| `desktop.resizable` | 真偽値（`true \| false`） | `true` | ユーザーによるデスクトップウィンドウのリサイズを許可 |
| `desktop.titleBarStyle` | `'default' \| 'hidden'` | `'default'` | デスクトップウィンドウのタイトルバー外観 |

<a name="config/view.js"></a>

### config/view.js

dframeworkはビューエンジンの設定用として任意の`config/view.js`設定ファイルをサポートしています。

```javascript
// config/view.js
import { Env } from 'dframework';

export default {
  maxYieldDepth: Env.value('VIEW_MAX_YIELD_DEPTH', 30),
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `maxYieldDepth` | `VIEW_MAX_YIELD_DEPTH` | 整数（正の整数） | `30` | ネストされた`@yield`セクション解決の最大再帰深度 |

<a name="config/tracking.js"></a>

### config/tracking.js

トラッキング設定はリクエストテレメトリモードおよびエクスポートシンクを制御します:

```javascript
// config/tracking.js
import { Env } from 'dframework';

export default {
  mode: Env.value('TRACKING_MODE', 'off'),
  sinks: {
    file: {
      enabled: Env.value('TRACKING_FILE_ENABLED', false),
      path: Env.value('TRACKING_FILE_PATH', 'storage/framework/tracking/requests.ndjson'),
    },
    otlp: {
      enabled: Env.value('TRACKING_OTLP_ENABLED', false),
      endpoint: Env.value('OTEL_EXPORTER_OTLP_ENDPOINT', 'http://localhost:4318/v1/traces'),
      serviceName: Env.value('OTEL_SERVICE_NAME', 'dframework-app'),
    },
  },
};
```

| キー | 環境変数 | 許容値 | デフォルト値 | 説明 |
| --- | --- | --- | --- | --- |
| `mode` | `TRACKING_MODE` | `'off' \| 'errors' \| 'all'` | `'off'` | グローバルリクエストテレメトリキャプチャモード |
| `sinks.file.enabled` | `TRACKING_FILE_ENABLED` | 真偽値（`true \| false`） | `false` | ndjsonファイルエクスポートシンクの有効化 |
| `sinks.file.path` | `TRACKING_FILE_PATH` | 文字列（ファイルパス） | `'storage/framework/tracking/requests.ndjson'` | ndjsonテレメトリログファイルのパス |
| `sinks.otlp.enabled` | `TRACKING_OTLP_ENABLED` | 真偽値（`true \| false`） | `false` | opentelemetry otlpトレースエクスポートシンクの有効化 |
| `sinks.otlp.endpoint` | `OTEL_EXPORTER_OTLP_ENDPOINT` | 文字列（url） | `'http://localhost:4318/v1/traces'` | otlpコレクターhttpエンドポイントurl |
| `sinks.otlp.serviceName` | `OTEL_SERVICE_NAME` | 文字列 | `'dframework-app'` | otlpスパンに付与されるサービス名属性 |

<a name="設定値へのアクセス"></a>

## 設定値へのアクセス

dframeworkはあらゆるコンテキストで利用可能なグローバルな`Config`ファサードを提供します。最初のセグメントが設定ファイル名、残りがそのファイル内のキーパスとなるドット記法を使用して値を取得します。

```javascript
const env = Config.get('app.env');
const dbName = Config.get('database.name');
const authModel = Config.get('auth.model', 'User');

if (Config.has('database.host')) {
  // configuration key is defined
}
```

`Config.get`の第2引数はキーが存在しない場合に返される任意のフォールバック値です。フォールバックが指定されておらずキーが見つからない場合、メソッドは`null`を返します。設定キーが定義されているかどうかを確認するには`Config.has`を使用します。

<a name="環境変数へのアクセス"></a>

## 環境変数へのアクセス

設定ファイル内では`Env.value()`を使用して解析された`.env`ファイルから値を読み取ります。設定システムを経由せずに生の環境変数の値が必要な場所であればどこでも使用できます。

```javascript
import { Env } from 'dframework';

const port = Env.value('APP_PORT', 8250);
const dbHost = Env.value('DB_HOST', 'localhost');
```

<a name="設定のホットリロード"></a>

## 設定のホットリロード

`local`環境ではフレームワークはファイル変更を検知するために`config`ディレクトリ全体を監視します。設定ファイルまたは`.env`を保存すると、ウォッチャーが変更を検知してサーバープロセスを再起動するため更新された設定値が即座に反映されます。
