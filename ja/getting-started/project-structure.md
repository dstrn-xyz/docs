# プロジェクト構造

- [はじめに](#はじめに)

- [ルートディレクトリ](#ルートディレクトリ)

  - [configディレクトリ](#configディレクトリ)
  - [consoleディレクトリ](#consoleディレクトリ)
  - [controllersディレクトリ](#controllersディレクトリ)
  - [databaseディレクトリ](#databaseディレクトリ)
  - [jobsディレクトリ](#jobsディレクトリ)
  - [langディレクトリ](#langディレクトリ)
  - [middlewaresディレクトリ](#middlewaresディレクトリ)
  - [modelsディレクトリ](#modelsディレクトリ)
  - [nativeディレクトリ](#nativeディレクトリ)
  - [publicディレクトリ](#publicディレクトリ)
  - [routesディレクトリ](#routesディレクトリ)
  - [storageディレクトリ](#storageディレクトリ)
  - [viewsディレクトリ](#viewsディレクトリ)

<a name="はじめに"></a>

## はじめに

dframeworkは決定論的なアプリケーション構造を強制します。デフォルトのディレクトリ構造を設定や変更することはできません。この厳格さによりフレームワークで構築されたすべてのアプリケーションがどの開発者にとっても即座に馴染み深いものとなり、内部エンジンがロードパスやコンパイルパスを積極的に最適化できるようになります。

> [!NOTE]
> このディレクトリ構造から逸脱しようとすると、ブートシーケンスまたはビューコンパイルフェーズでフレームワークエンジンが失敗します。

<a name="ルートディレクトリ"></a>

## ルートディレクトリ

<a name="configディレクトリ"></a>

### configディレクトリ

`config`ディレクトリにはすべての設定ファイルが保持されます。各ファイルはファイル名の名前空間にマッピングされたデフォルトオブジェクトをエクスポートします。

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

<br />

<a name="consoleディレクトリ"></a>

### consoleディレクトリ

バックグラウンドタスクとスケジューリング定義は`console`ディレクトリに配置されます。フレームワークは定期実行ジョブやバックグラウンドワーカーを設定するために、このディレクトリにあるスケジュール定義ファイルを特定して参照します。カスタムコマンドもここで登録されます。

```javascript
// console/Schedule.js
export default function (scheduler) {
  scheduler.command('CleanupExpiredSessions').daily();
}
```

<a name="controllersディレクトリ"></a>

### controllersディレクトリ

`controllers`ディレクトリには受信リクエストロジックを処理するクラスが含まれます。コントローラーはルーターによって自動的に呼び出されるメソッドへと振る舞いをまとめます。継承すべきベースコントローラークラスは存在しません。ロジックで必要としない場合、リクエストやレスポンスオブジェクトなどの引数は完全に任意です。

```javascript
// controllers/IndexController.js
export default class IndexController {
  index() {
    return render('index');
  }
}
```

<a name="databaseディレクトリ"></a>

### databaseディレクトリ

`database`ディレクトリにはマイグレーションやシーダーを含むすべてのデータベース関連ファイルが保持されます。フレームワークはスキーマ構築コマンドを実行する際にこのディレクトリを利用し、環境間でのデータベースの変更追跡を可能にします。

```javascript
// database/migrations/2026_01_01_000000_create_users_table.js
export async function up() {
  await Schema.create('users', table => {
    table.increments();
    table.string('email').unique();
    table.string('password');
    table.timestamps();
  });
}

export async function down() {
  await Schema.dropIfExists('users');
}
```

```javascript
// database/seeders/DatabaseSeeder.js
export async function run() {
  await table('users').insert({
    email: 'admin@example.com',
    password: 'password'
  });
}
```

<a name="jobsディレクトリ"></a>

### jobsディレクトリ

`jobs`ディレクトリにはワーカースレッドによって非同期に実行されるバックグラウンドジョブクラスが配置されます。すべてのジョブクラスは`handle(payload)`メソッドを持つデフォルトクラスをエクスポートし、アプリケーション全体で`Job.dispatch()`を介してディスパッチされます。

```javascript
// jobs/SendWelcomeEmailJob.js
export default class SendWelcomeEmailJob {
  async handle(payload) {
    const user = await User.find(payload.userId);
    await Mailer.send(user.email, 'welcome', { name: user.name });
  }
}
```

<a name="langディレクトリ"></a>

### langディレクトリ

ローカライゼーション文字列と翻訳辞書は`lang`ディレクトリに保存されます。これらは通常ロケール固有のフォルダ内にJSONファイルとして配置されます。フレームワークはこれらのファイルを自動的にロードし、アプリケーションおよびビューテンプレート全体で多言語サポートを提供します。

```json
// lang/en/common.json
{
  "welcome": "welcome to dframework",
  "login": "log in to your account"
}
```

<a name="middlewaresディレクトリ"></a>

### middlewaresディレクトリ

`middlewares`ディレクトリにはアプリケーションに入るHTTPリクエストをフィルタリングおよび検査するクラスが含まれます。コントローラーに到達する前に認証、レートリミット、リクエスト操作を強制するカスタムロジックをここで定義します。

```javascript
// middlewares/EnsureAuth.js
export default class EnsureAuth {
  handle(req, next) {
    if (!Auth.check()) return abort(401);
    return next(req);
  }
}
```

<a name="modelsディレクトリ"></a>

### modelsディレクトリ

Active Recordエンティティは`models`ディレクトリに配置されます。フレームワークは認証フローや暗黙的なルートモデルバインディングなどでモデルを動的に解決する際にこの正確な場所に依存します。モデルはデータベーススキーマ抽象化の唯一の信頼できる情報源を表します。

```javascript
// models/User.js
export default class User extends Model {
  // model definition
}
```

<a name="nativeディレクトリ"></a>

### nativeディレクトリ

`native`ディレクトリにはWebアプリケーションをネイティブモバイルおよびデスクトップ環境に統合するために特異的に必要なコードが含まれます。フレームワークはこのディレクトリを利用してネイティブコンパイルとアセット注入を処理し、WebViewとネイティブデバイス機能を橋渡しします。

<a name="publicディレクトリ"></a>

### publicディレクトリ

静的アセットや一般公開ファイルは`public`ディレクトリに配置されます。内部Webサーバーはローカル開発中にこれらのファイルを直接提供できます。本番環境ではこのディレクトリがWebサーバー構成のドキュメントルートとして機能します。

<a name="routesディレクトリ"></a>

### routesディレクトリ

`routes`ディレクトリはすべての受信リクエストのエントリポイントです。フレームワークはこのフォルダ内にあるすべてのルーティング定義を自動的に読み込んで登録します。ここでWebエンドポイントとアプリケーションロジックのマッピングを定義します。フレームワークが起動プロセス中に自動検出を処理するため、これらのファイルを他の場所で手動インポートする必要はありません。

```javascript
// routes/web.js
Route.get('/', 'HomeController@index');
```

<a name="storageディレクトリ"></a>

### storageディレクトリ

`storage`ディレクトリはアプリケーションによって生成されたファイル用に予約されています。これにはキャッシュされたビュー、一時的なホットリロードデータ、ユーザーがアップロードしたコンテンツが含まれます。フレームワークはこのディレクトリの内部構造を自動的に管理し、公開アクセス可能なアップロードファイル用のシンボリックリンクを維持します。

<a name="viewsディレクトリ"></a>

### viewsディレクトリ

`views`ディレクトリにはすべてのフロントエンドテンプレートが含まれます。内部ビューコンパイラは本番ビルドパイプライン中にこれらのテンプレートを静的に解析して最適化します。コンパイルエンジンはこのパスを明示的なターゲットとしているため、テンプレートをこのディレクトリの外に配置することはできません。

```blade
@extends('layouts.app')

@section('content')
  <div>welcome to dframework</div>
@endsection
```
