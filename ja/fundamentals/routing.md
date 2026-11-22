# ルーティング

- [ルーティング](#ルーティング)
  - [はじめに](#はじめに)
  - [基本的なルート](#基本的なルート)
    - [利用可能なメソッド](#利用可能なメソッド)
  - [ルートパラメータ](#ルートパラメータ)
    - [パラメータの制約](#パラメータの制約)
  - [名前付きルート](#名前付きルート)
  - [ルートグループ](#ルートグループ)
    - [プレフィックス](#プレフィックス)
    - [ミドルウェア](#ミドルウェア)
    - [複合属性](#複合属性)
    - [シールドとcsrfのグループ単位での無効化](#シールドとcsrfのグループ単位での無効化)
    - [ミドルウェアの実行順序](#ミドルウェアの実行順序)
  - [ドメインとポートによるルーティング](#ドメインとポートによるルーティング)
    - [ドメインルーティング](#ドメインルーティング)
    - [ポートルーティング](#ポートルーティング)
    - [チェーン可能なビルダー](#チェーン可能なビルダー)
    - [パラメータ付きドメイン](#パラメータ付きドメイン)
    - [複数ドメイン間でのルート名の重複](#複数ドメイン間でのルート名の重複)
  - [インラインミドルウェアチェーン](#インラインミドルウェアチェーン)
  - [ルート修飾子](#ルート修飾子)
    - [csrf](#csrf)
    - [シールド](#シールド)
    - [ルートレベルミドルウェア](#ルートレベルミドルウェア)
    - [ログ](#ログ)
    - [トラッキング](#トラッキング)
    - [ビュー](#ビュー)
    - [プロファイル](#プロファイル)
    - [ベーシック認証](#ベーシック認証)
  - [コントローラ文字列の構文](#コントローラ文字列の構文)
  - [複数のルートファイル](#複数のルートファイル)

<a name="はじめに"></a>

## はじめに

ルートは`routes`ディレクトリ内で定義します。dframeworkは手動でのインポートを必要とせず起動時にそのディレクトリ内のすべての`.js`ファイルを自動的に検出して読み込みます。各ルートファイル内では内部ルーターインスタンスへ直接プロキシするグローバルな`Route`ファサードを利用できます。

<a name="基本的なルート"></a>

## 基本的なルート

最もシンプルなルートはurlパスと処理を担当するメソッドを指すコントローラ文字列を受け取ります。

```javascript
// routes/web.js
Route.get('/dashboard', 'app.IndexController@dashboard');
Route.post('/register', 'auth.AuthController@register');
Route.put('/profile', 'app.UserProfileController@update');
Route.patch('/profile/avatar', 'app.UserProfileController@updateAvatar');
Route.delete('/session', 'auth.AuthController@logout');
```

<a name="利用可能なメソッド"></a>

### 利用可能なメソッド

ルーターは標準的なhttp動詞をサポートしています。

```javascript
Route.get(path, handler);
Route.post(path, handler);
Route.put(path, handler);
Route.patch(path, handler);
Route.delete(path, handler);
Route.head(path, handler);
```

<a name="ルートパラメータ"></a>

## ルートパラメータ

動的なurlセグメントには先頭にコロンを付与します。ルーターはこれらの値を抽出およびデコードしコントローラ内の`req.params`で利用できるようにします。

```javascript
Route.get('/user/:id', 'app.IndexController@user');
Route.get('/admin/:entity/:id/edit', 'admin.AdminController@editForm');
```

```javascript
// controllers/app/IndexController.js
export default class IndexController {
  async user(req) {
    const user = await User.find(req.params.id);
    return json({ user });
  }
}
```

<a name="パラメータの制約"></a>

### パラメータの制約

ルート定義で`.where()`メソッドを使用してルートパラメータの形式を制約できます。`.where()`メソッドはパラメータ名と正規表現パターン文字列、数値、または制約オブジェクトを受け取ります。数値の制約は完全一致の正規表現へと自動的にコンパイルされます:

```javascript
Route.get('/user/:id', 'app.IndexController@user').where('id', '[0-9]+');
Route.get('/user/:name', 'app.IndexController@show').where('name', '[a-zA-Z]+');

// 数値は完全一致の正規表現にコンパイルされます
Route.get('/special/:id', 'app.IndexController@special').where('id', 42);

// 複数パラメータのオブジェクト構文
Route.get('/posts/:year/:slug', 'app.PostController@show').where({
  year: '^[0-9]{4}$',
  slug: '^[a-z0-9-]+$'
});
```

<a name="名前付きルート"></a>

## 名前付きルート

ルート定義の後に`.name()`をチェーンすることでルートに名前を付けます。名前付きルートを使用するとパスをハードコードすることなくプログラム上でurlを生成できます。

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard').name('dashboard');
Route.get('/user/:id', 'app.IndexController@user').name('app.user.profile');
```

名前付きルートからurlを生成するにはグローバルな`route()`ヘルパーを使用します。

```javascript
const dashboardUrl = route('dashboard');
const profileUrl = route('app.user.profile', { id: 42 });
```

<a name="ルートグループ"></a>

## ルートグループ

グループを使用すると個別の定義で繰り返すことなく複数のルート間でプレフィックスやミドルウェアなどの属性を共有できます。

<a name="プレフィックス"></a>

### プレフィックス

```javascript
Route.group({ prefix: '/faq' }, (faq) => {
  faq.get('/', 'app.IndexController@faq').name('app.faq');
  faq.get('/tos', 'app.IndexController@faq_tos').name('app.faq.tos');
  faq.get('/privacy', 'app.IndexController@faq_privacy').name('app.faq.privacy');
});
```

<a name="ミドルウェア"></a>

### ミドルウェア

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.get('/home', 'app.HomeController@home').name('app.home');
  auth.get('/library', 'app.IndexController@library').name('app.library');
});
```

<a name="複合属性"></a>

### 複合属性

プレフィックス、ミドルウェア、shield、csrfオプションは単一のグループ定義内でまとめて設定できます。

```javascript
Route.group({
  prefix: '/admin',
  middleware: ['AuthMiddleware@requireAuth', 'AdminMiddleware@requireAdmin'],
  shield: true,
  csrf: true
}, (admin) => {
  admin.get('/', 'admin.AdminController@index').name('admin.index');
  admin.get('/:entity', 'admin.AdminController@list').name('admin.list');
  admin.post('/:entity', 'admin.AdminController@store').name('admin.store');
});
```

<a name="シールドとcsrfのグループ単位での無効化"></a>

### シールドとcsrfのグループ単位での無効化

webhookエンドポイント、公開api、またはサードパーティからのコールバック用にはグループ内のすべてのルートでshieldおよびcsrf検証を無効化できます:

```javascript
Route.group({ prefix: '/webhooks', csrf: false, shield: false }, (hooks) => {
  hooks.post('/stripe', 'StripeController@handle');
  hooks.post('/github', 'GitHubController@handle');
});
```

流暢なチェーン可能ビルダーを使用することもできます:

```javascript
Route.prefix('/webhooks')
  .shield(false)
  .csrf(false)
  .group((hooks) => {
    hooks.post('/stripe', 'StripeController@handle');
  });
```

グループはネスト可能です。内側のグループは親からshieldおよびcsrfの設定を継承し必要に応じて明示的に上書きできます。

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.group({ prefix: '/api/preferences' }, (prefs) => {
    prefs.get('/', 'app.PreferencesController@index').name('api.preferences.index');
    prefs.put('/:key', 'app.PreferencesController@update').name('api.preferences.update');
  });
});
```

<a name="ミドルウェアの実行順序"></a>

### ミドルウェアの実行順序

ルートが1つ以上のグループ内に存在する場合実行時のミドルウェアスタックは`外側グループのミドルウェア, 内側グループのミドルウェア, ルートレベルのミドルウェア, コントローラ`となります。グループミドルウェアは登録時に宣言順でルート自身のハンドラーチェーンの先頭に追加されるためネストされたグループは自身のミドルウェアより前に親のミドルウェアを実行します。

```javascript
Route.group({ middleware: [a] }, (g1) => {
  g1.group({ middleware: [b] }, (g2) => {
    g2.middleware(c).get('/path', 'Ctrl@index');
  });
});
// runtime stack for GET /path: [a, b, c, Ctrl@index]
```

`next()`を呼び出さずにレスポンスを返すミドルウェア（リクエストを拒否する組み込みの`RateLimiter`など）はチェーンの後続処理をショートサーキットします。コントローラは実行されず後続のミドルウェアも一切実行されません。これはグループとその内部のルートの両方に`RateLimiter`を適用する場合に特に重要です。2つのリミッターは協調動作しません。それぞれが独自のウィンドウとカウントを追跡し先に上限に達した方が優先されます。また両方の層で同一の`RateLimiter`インスタンスを再利用するとリクエストごとの加算が2倍になります（そのため実質的な制限値は`max / 2`となります）。詳細な連携モデルについてはセキュリティドキュメントの[レイヤードレートリミッター](../security/security.md#layered-rate-limiters-group-plus-route)セクションを参照してください。

<a name="ドメインとポートによるルーティング"></a>

## ドメインとポートによるルーティング

dframeworkはドメイン別およびポート別のルート登録をサポートしています。グループ属性またはチェーン可能なビルダーメソッドのいずれかを使用して特定のホスト、サブドメイン、ポートにルートを制限できます。

<a name="ドメインルーティング"></a>

### ドメインルーティング

特定のホストまたはサブドメインにルートを制限するには`Route.group()`に`domain`オプションを渡すか`Route.domain()`を呼び出します。

```javascript
Route.group({ domain: 'admin.example.com' }, (admin) => {
  admin.get('/dashboard', 'admin.AdminController@index').name('admin.dashboard');
});

Route.domain('api.example.com', (api) => {
  api.get('/v1/users', 'api.UserController@index').name('api.users');
});
```

<a name="ポートルーティング"></a>

### ポートルーティング

`port`オプションまたは`Route.port()`を使用して特定のリスニングポートにルートを制約することもできます。ポートの一致判定はhostヘッダーまたはローカルサーバーソケットポートをチェックします。

```javascript
Route.group({ port: 8080 }, (metrics) => {
  metrics.get('/health', 'app.HealthController@check');
});

Route.port(9090, (internal) => {
  internal.get('/metrics', 'app.MetricsController@export');
});
```

<a name="チェーン可能なビルダー"></a>

### チェーン可能なビルダー

`.domain()`、`.port()`、`.basicAuth()`、`.middleware()`は完全にチェーン可能なビルダーでありエンドポイントやグループを登録する前に任意の順序で構成できます。

```javascript
Route.domain('shop.example.com').get('/cart', 'shop.CartController@index');

Route.basicAuth('admin', 'secret')
  .port(8080)
  .get('/metrics', 'app.MetricsController@export');

Route.domain('secure.example.com')
  .port(8443)
  .middleware('AuthMiddleware@requireAuth')
  .get('/data', 'app.DataController@show');

Route.domain('admin.example.com')
  .port(8080)
  .group({ prefix: '/v1' }, (v1) => {
    v1.get('/users', 'admin.UserController@index');
  });
```

<a name="パラメータ付きドメイン"></a>

### パラメータ付きドメイン

ドメイン定義は`:param`または`{param}`の構文を使用した動的パラメータプレースホルダーをサポートします。抽出されたドメインパラメータはパスパラメータとともに`req.params`で自動的に利用可能になります。

```javascript
Route.domain(':subdomain.example.com', (tenant) => {
  tenant.get('/users/:id', 'app.TenantController@showUser');
});

Route.domain('{tenant}.myapp.com', (app) => {
  app.get('/settings', 'app.SettingsController@show');
});
```

```javascript
// GET http://acme.example.com/users/42
// req.params.subdomain === 'acme'
// req.params.id === '42'
```

<a name="複数ドメイン間でのルート名の重複"></a>

### 複数ドメイン間でのルート名の重複

> [!WARNING]
> `.name()`で登録されたルート名はグローバルルックアップテーブルに保持されます。異なるドメイン上の2つのルートがまったく同一の名前を共有している場合後からの定義が以前のエントリを上書きします。常にルート名に対象ドメインまたはセクション名をプレフィックスとして付与してください（例: `admin.dashboard`や`app.dashboard`）。

```javascript
Route.domain('admin.example.com').get('/dashboard', 'AdminController@dash').name('admin.dashboard');
Route.domain('app.example.com').get('/dashboard', 'AppController@dash').name('app.dashboard');

// route('admin.dashboard') // '//admin.example.com/dashboard'
// route('app.dashboard') // '//app.example.com/dashboard'
```

`route(name, params)`を使用してドメインバインドされたルートのurlを生成する場合フレームワークは提供された`params`オブジェクトから任意のドメインパラメータを埋め込みプロトコル相対絶対url（`//admin.example.com/dashboard`など）を自動的に返します。

<a name="インラインミドルウェアチェーン"></a>

## インラインミドルウェアチェーン

単一のコントローラ文字列の代わりにハンドラーとして配列を渡すことができます。フレームワークは最後の要素より前のすべてのエントリをミドルウェアとして扱い最後の要素をコントローラとして処理します。

```javascript
Route.post('/approve', ['AuthMiddleware@requireAuth', 'auth.AuthController@approvePost']).name('approve.post');
```

グループを使用せずにルート定義の前または後に`.middleware()`を呼び出すことで単一のルートにミドルウェアを適用することもできます。

```javascript
// prefixed builder
Route.middleware(authLimiter).post('/register', 'auth.AuthController@register');

// or chained directly on the route definition
Route.post('/register', 'auth.AuthController@register').middleware(authLimiter);
```

<a name="ルート修飾子"></a>

## ルート修飾子

すべてのルート定義は特定のルート上でフレームワークの動作を制御するいくつかの修飾子を公開するチェーン可能オブジェクトを返します。

<a name="csrf"></a>

### csrf

csrf検証はすべての`post`、`put`、`patch`、`delete`ルートでデフォルトで有効になっています。`.csrf(false)`をチェーンすることで特定のルートに対して無効化できます。

```javascript
Route.post('/magic/:token', 'auth.AuthController@magicConsume').csrf(false);

Route.csrf(false, (hooks) => {
  hooks.post('/stripe/webhook', 'StripeController@handle');
});
```

<a name="シールド"></a>

### シールド

shieldセキュリティシステムがアクティブな場合受信したミューテーションリクエストを検証します。`.shield(false)`をチェーンするかルートを`Route.shield(false, callback)`で囲むことでルートを除外できます。

```javascript
Route.post('/magic/:token', 'auth.AuthController@magicConsume').shield(false);

Route.shield(false, (publicApi) => {
  publicApi.post('/api/external', 'ExternalController@handle');
});
```

<a name="ルートレベルミドルウェア"></a>

### ルートレベルミドルウェア

任意のルート定義に直接`.middleware()`をチェーンしてルートレベルミドルウェアをアタッチできます。ミドルウェア文字列、ミドルウェアの配列、または`RateLimiter`インスタンスを受け付けます。複数の`.middleware()`呼び出しを順番にチェーンすることもできます。

```javascript
Route.get('/profile', 'ProfileController@show').middleware('AuthMiddleware@requireAuth');

Route.post('/orders', 'OrderController@store').middleware(['AuthMiddleware@requireAuth', orderLimiter]);

Route.get('/settings', 'SettingsController@index')
  .middleware('AuthMiddleware@requireAuth')
  .middleware('TwoFactorMiddleware@require2fa');
```

<a name="ログ"></a>

### ログ

リクエストとレスポンスのロギングは`local`環境では自動的に行われます。本番環境で特定のルートのロギングを強制するには`.log()`をチェーンします。

```javascript
Route.post('/api/playback/track', 'app.PlaybackController@get_track').name('api.playback.track').log();
```

<a name="トラッキング"></a>

### トラッキング

`.track()`修飾子を使用すると一意のuuid関連付けidによる高パフォーマンスなリクエストトラッキングが有効になります。トラッキングが有効な場合、リクエストidが自動生成され、`X-Request-Id`レスポンスヘッダーが設定され、データベースクエリとログ出力が関連付けられ、エラービューやjsonエラーペイロードにトラッキングidが含まれます。

```javascript
Route.post('/checkout', 'CheckoutController@process').track();
```

ルートグループ全体でトラッキングを有効にすることもできます:

```javascript
Route.group({ track: true }, () => {
  Route.post('/payments/charge', 'PaymentController@charge');
  Route.get('/payments/status', 'PaymentController@status');
});
```

または`Route.track`を使用してコールバックブロックをラップします:

```javascript
Route.track(true, (group) => {
  group.post('/payments/charge', 'PaymentController@charge');
  group.get('/payments/status', 'PaymentController@status');
});
```

<a name="ビュー"></a>

### ビュー

`.views()`修飾子はルートコンパイラに対してこのルートがレンダリングするビューテンプレートを正確に伝えます。本番ビルド中にコンパイラはそれらのテンプレートを解析してコンパイルされたハンドラーが実際に必要とするセッションデータ、ロケール検出、リクエストボディ解析を特定します。明示的なビュー名の指定は静的解析ステップにおける曖昧さを排除する最適化ヒントとなります。

動的なビュー名（変数を介したレンダリングなど）はコンパイラによって静的に検査できません。パフォーマンスが重要なルートでは動的なビュー名は推奨されません。動的ビューのレンダリングが必要な場合はコンパイラが翻訳タグ、レイアウト依存関係、ミドルウェアの要件を検出できるように考えられるすべてのビュー名を指定して`.views(['template.name'])`を提供する必要があります。

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard').name('dashboard').views(['app.home.index']);
```

<a name="プロファイル"></a>

### プロファイル

`.profile()`修飾子を使用するとコンパイラがルートの静的解析から導出した動作プロファイルを上書きできます。解析された値を置き換えるキーを持つオブジェクトを渡します。これは自動解析によって誤ったまたは最適ではないコンパイル済みハンドラーが生成されるルートに対する高度なエスケープハッチです。

利用可能なプロファイルキーは以下の通りです:

- `needsSession`: boolean
- `needsPreviousUrl`: boolean
- `needsFlash`: boolean
- `needsBody`: boolean
- `needsLocale`: boolean
- `hasCsrf`: boolean
- `hasShield`: boolean

```javascript
Route.get('/feed', 'app.FeedController@index').profile({ needsSession: true, needsFlash: true });
```

`.csrf(false)`と`.shield(false)`の両方が指定された変更系ルート（`POST`、`PUT`、`PATCH`、`DELETE`）では、本番用のルートコンパイラは静的解析によってボディへのアクセス（`req.body`、`req.files`、`req.file()`、`req.hasFile()`、`req.input()`、`req.all()`、または `validate()`）が検出された場合にのみリクエストボディをパースします。`req`を`Model`のメソッドに直接渡すパターン（`await Post.createFromRequest(req)`）、`body`や`files`の前にネストされた分割代入を行うパターン（`async update({ params: { id }, body })`）、または`.input()`や`.all()`を呼び出す前に`req`をローカル変数にエイリアスするパターン（`const r = req; r.input('name')`）といった間接的な書き方は解析器がボディアクセスとして認識しないため推奨されません。csrfとshieldを無効にしたルートでこれらのパターンを意図的に使用する場合は、`.profile({ needsBody: true })`を指定してボディのパースを明示的に強制してください:

```javascript
Route.post('/api/posts/:id', 'ApiPostController@update')
  .csrf(false)
  .shield(false)
  .profile({ needsBody: true });
```

> [!NOTE]
> ローカル開発環境では変更系ルートは常に無条件でリクエストボディをパースします（不正なjsonの場合は`400`、`app.maxBodySize`を超えるペイロードの場合は`413`を返します）。一方、本番環境では`.csrf(false)`と`.shield(false)`が指定されておりボディへのアクセスも`.profile({ needsBody: true })`もないコンパイル済みルートはボディのパースを完全にスキップするため、それらのエンドポイントに不正なjsonやサイズ超過のボディが送信されても自動的な`400`または`413`レスポンスは返されません。

<a name="ベーシック認証"></a>

### ベーシック認証

`.basicAuth()`をチェーンして特定のルートをhttpベーシック認証で保護できます。明示的な認証情報、カスタムバリデータコールバック、カスタムrealmを含むオプションオブジェクトを受け入れるか`config/auth.js`（`auth.basic`）で設定されたデフォルト値を使用します。

```javascript
Route.get('/secret', 'app.SecretController@show').basicAuth('admin', 'secret');
Route.get('/admin', 'app.AdminController@index').basicAuth();
Route.get('/users', 'app.UserController@index').basicAuth(async (user, pass) => user === 'admin' && pass === 'secret');
```

<a name="コントローラ文字列の構文"></a>

## コントローラ文字列の構文

コントローラ文字列は`directory.ControllerName@method`の規則に従います。ディレクトリセグメントは`controllers/`内のサブディレクトリにマッピングされます。`controllers/app/IndexController.js`にあるコントローラは`app.IndexController@index`として参照されます。`controllers/auth/AuthController.js`にあるコントローラは`auth.AuthController@register`として参照されます。

```javascript
Route.get('/dashboard', 'app.IndexController@dashboard');
Route.post('/register', 'auth.AuthController@register');
```

サブディレクトリがなく`controllers/`ルートに直接配置されたコントローラの場合はディレクトリプレフィックスを完全に省略します。

```javascript
Route.get('/locale/:locale', 'LocaleController@set');
```

<a name="複数のルートファイル"></a>

## 複数のルートファイル

`routes`ディレクトリ内で必要に応じてルートを複数のファイルに分割できます。フレームワークはその場所で見つかったすべての`.js`ファイルを自動的に読み込みます。読み込みの順序はファイルシステムのソート順に従います。

```javascript
// routes/web.js (http routes)
// routes/wire.js (socket routes)
```
