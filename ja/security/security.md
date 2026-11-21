# セキュリティ

- [セキュリティ](#セキュリティ)
  - [CSRF保護](#CSRF保護)
    - [トークンのローテーション](#トークンのローテーション)
  - [レート制限](#レート制限)
    - [設定](#設定)
    - [ユーザーごとのレート制限](#ユーザーごとのレート制限)
    - [信頼できるプロキシ](#信頼できるプロキシ)
    - [メモリ上限とLRUエビクション](#メモリ上限とLRUエビクション)
    - [階層化レートリミッター（グループとルート）](#階層化レートリミッター（グループとルート）)
  - [基本認証](#基本認証)
    - [ルートの保護](#ルートの保護)
  - [ShieldGuard](#ShieldGuard)

<a name="CSRF保護"></a>

## CSRF保護

dframeworkはアプリケーションによって管理されるアクティブな各ユーザーセッションに対してクロスサイトリクエストフォージェリ（csrf）トークンを自動的に生成します。このトークンは認証されたユーザーが実際にアプリケーションに対してリクエストを行っている本人であることを検証するために使用されます。

csrfトークンは暗号学的に安全であり`APP_KEY`を使用して生成されたhmacに依存します。

```javascript
// generate a new token
const token = req.csrfToken();

// verify a submitted token
if (req.verifyCsrfToken(submittedToken)) {
  // request is safe
}
```

### ルートおよびグループでのCSRFの無効化

セッショントークンを提供できない公開webhookや外部コールバックのために個別ルートまたはグループ全体でcsrf検証を無効化できます。

```javascript
// single route
Route.post('/webhooks/stripe', 'WebhookController@handle').csrf(false);

// route group
Route.group({ prefix: '/webhooks', csrf: false }, (hooks) => {
  hooks.post('/stripe', 'StripeController@handle');
  hooks.post('/github', 'GitHubController@handle');
});

// fluent chaining
Route.prefix('/webhooks').csrf(false).group((hooks) => {
  hooks.post('/stripe', 'StripeController@handle');
});
```

<a name="トークンのローテーション"></a>

### トークンのローテーション

フレームワークによって生成されたcsrfトークンは6時間のアクティブ期間中有効であり専用のクッキーに関連付けられます。手動でトークンをローテーションしそのクライアントの以前のトークンを無効化する必要がある場合は`rotateCsrfToken`メソッドを使用できます。セッションの再生成時にもトークンは自動的にローテーションされます。

<a name="レート制限"></a>

## レート制限

dframeworkには不正利用やブルートフォース攻撃を防止するために設計されたスライディングウィンドウレートリミッターが含まれています。クライアントが指定された時間枠内の最大許容リクエスト数を超過した場合フレームワークは自動的にhttp 429「too many requests」ステータスコードを返します。

```javascript
import { RateLimiter } from 'dframework';

const limiter = new RateLimiter({
  windowMs: 60000,     // time window in milliseconds (default 60000 = 1 minute)
  max: 60,             // maximum requests per window (default 60)
  message: 'too many requests'
});

// apply to a single route
Route.middleware(limiter).post('/register', 'auth.AuthController@register');

// apply to a route group
Route.middleware(limiter).group({ prefix: '/api' }, (api) => {
  api.get('/status', 'api.StatusController@index');
});
```

<a name="設定"></a>

### 設定

`RateLimiter`コンストラクターは以下のオプションを受け付けます。すべての項目は適切なデフォルト値を持つオプションです。

| オプション | 型 | デフォルト値 | 説明 |
| --- | --- | --- | --- |
| `windowMs` | `number` | `60000`（1分） | ミリ秒単位のスライディングウィンドウのサイズ |
| `max` | `number` | `60` | 単一ウィンドウ内で許可される最大リクエスト数 |
| `message` | `string` | `'too many requests'` | 制限を超過した際にjsonボディで返されるエラーメッセージ |
| `keyGenerator` | `Function` | `(req) => req.ip` | 追跡する文字列キーを返す関数（ユーザーごとやapiトークンごとに制限するようカスタマイズ可能） |
| `trustedProxies` | `string[]` | `[]` | `x-forwarded-for`を解決するための信頼できるプロキシIPアドレスの配列 |
| `maxEntries` | `number` | `100000` | lruエビクション前にメモリに保持される追跡対象クライアントキーの最大個別数 |
| `headers` | `boolean` | `false` | 上限未超過のリクエストでレート制限クォータヘッダーを送信するかどうか（状態秘匿のためデフォルトで無効） |

<a name="ユーザーごとのレート制限"></a>

### ユーザーごとのレート制限

IPアドレスの代わりに認証済みユーザーごとにレート制限を追跡するにはカスタムの`keyGenerator`を使用します。未認証の訪問者はリモートアドレスにフォールバックします。

```javascript
const perUserLimiter = new RateLimiter({
  windowMs: 60000,
  max: 5,
  message: 'too many login attempts',
  keyGenerator: (req) => Auth.id() || req.ip
});

Route.middleware(perUserLimiter).post('/login', 'auth.AuthController@login');
```

<a name="信頼できるプロキシ"></a>

### 信頼できるプロキシ

アプリケーションがロードバランサーやリバースプロキシの背後にある場合は`trustedProxies`配列を指定します。リミッターは`x-forwarded-for`チェーンを右から左へ辿り最も右側にある信頼できないIPをクライアントとして使用します。チェーン内のすべてのエントリが信頼されている場合（またはヘッダーが存在しない場合）は`req.ip`にフォールバックします。

```javascript
const apiLimiter = new RateLimiter({
  trustedProxies: ['10.0.0.1', '10.0.0.2'],
});
```

<a name="メモリ上限とLRUエビクション"></a>

### メモリ上限とLRUエビクション

各`RateLimiter`は最近最も使われていないマップ（`LruMap`）を使用してメモリ内でクライアントのリクエスト数を追跡します。各エントリは特定のクライアントキーに対する現在のヒットカウントとウィンドウ開始タイムスタンプを記録します。

数千のアクティブクライアントが存在する通常のトラフィック下ではデフォルトの`maxEntries`である`100000`が最小限のヒープメモリ消費（約数メガバイト）で十分なキャパシティを提供します。

**メモリ保護**
追跡対象キーの上限がない場合分散トラフィックの急増、ボット攻撃、ローテーションするクライアントアドレスによってnodeプロセスがメモリ不足エラーでクラッシュするまで新しいエントリが割り当てられ続ける可能性があります。`maxEntries`はサーバーにアクセスするクライアントキーの数に関係なくメモリ消費に厳格な上限を設定します。

**エビクションの挙動と調整**
アクティブな追跡対象キーの数が`maxEntries`に達すると新しい呼び出し元のための領域を確保するために最も長い間アクセスされていないキーが即座に追い出されます。アクティブなクライアントキーがその`windowMs`の期限切れ前に追い出された場合そのヒットカウンターは破棄されます。クライアントの次のリクエストは新しいウィンドウを開始するため意図せずレート制限カウントがリセットされることになります。

**高トラフィックアプリケーション**
単一ウィンドウ内で100000を超えるアクティブな一意のクライアントが予想される公開apiやマイクロサービスでは早期のエビクションを防ぐために高い`maxEntries`（`250000`や`500000`など）を設定してください。

**メモリ制約のある環境**
メモリ制限が厳しいサーバーや呼び出し元のプールが小さいと分かっているエンドポイントではヒープフットプリントを小さく維持するために`maxEntries`を低く設定します。

リミッターは標準のレート制限ヘッダー（`Retry-After`、`X-RateLimit-Limit`、`X-RateLimit-Remaining`）を各レスポンスに注入します。古いエントリを追い出すための内部クリーンアップ間隔がウィンドウごとに1回実行されます。アプリケーションのシャットダウン時にはこの間隔が停止されすべての追跡データが自動的にクリアされます。

<a name="階層化レートリミッター（グループとルート）"></a>

### 階層化レートリミッター（グループとルート）

ルートグループが`RateLimiter`を適用しそのグループ内のルートが2つ目の`RateLimiter`を適用する場合内部ルートへのすべてのリクエストで両方のリミッターが実行されます。フレームワークが一方を選択したり他方で置き換えたりすることはありません。

呼び出し順序はルート登録時に決定されます。グループミドルウェアがルート自体のハンドラーチェーンの先頭に追加されるためグループのリミッターが最初に実行されルートのリミッターが2番目に実行されます（その後にコントローラーが続きます）。実行時スタックは事実上`[groupLimiter, routeLimiter, controllerHandler]`となります。

```javascript
Route.group({ middleware: [apiLimiter] }, (api) => {
  api.middleware(strictLimiter).post('/login', 'AuthController@login');
});
// runtime stack per /login request: [apiLimiter, strictLimiter, AuthController@login]
```

各`RateLimiter`インスタンスは独自のスライディングウィンドウと独自のヒットカウントを維持します。2つのレイヤーは状態を共有しません。

**カウントは独立して累積されます**
同じIPごとのキーを使用する`apiLimiter({ max: 100 })`と`strictLimiter({ max: 5 })`の場合内部ルートは実質的にグループレベルで毎分100リクエスト、ルートレベルで毎分5リクエストを許可します。実際の呼び出し元が最初に到達するのはより厳格な制限（`strictLimiter`）です。

**最初に制限に達した方がチェーンを短絡させます**
リミッターがリクエストを拒否した場合`429`を送信し`next()`を呼び出さないため2番目のリミッターとコントローラーは実行されません。レスポンスには拒否したリミッターのヘッダー（`Retry-After`、`X-RateLimit-Limit`、`X-RateLimit-Remaining`）のみが含まれもう一方のリミッターがそれらを設定または上書きすることはありません。

**成功時には両方のリミッターがインクリメントされます**
リミッターがリクエストを通過させると自身のカウンターをインクリメントし`next()`を呼び出します。したがって両方のレイヤーを通過したリクエストは両方のバケットのカウントを増加させます。

> [!WARNING]
> グループと内部ルートの両方で同じ`RateLimiter`インスタンスを再利用しないでください。インスタンスを共有すると各リクエストでヒットカウントが2回インクリメントされ実効レート制限が`max / 2`に半減します。レイヤーごとに個別のリミッターインスタンスを作成してください。

すでにレート制限が設定されているグループ内の特定ルートに対してより厳格な上限が必要な場合はルートレイヤーで既存のIPごとの制限を強化する（より小さい`max`、`req.user.id`を含むカスタム`keyGenerator`、またはその両方）かそのルートに対してグループリミッターを解除することを推奨します。同じキーを持つ2つのリミッターを階層化すると調整なしにインクリメントが交差するため予期しないバジェット動作を引き起こす傾向があります。

<a name="基本認証"></a>

## 基本認証

dframeworkはhttp基本認証（basic authentication）の組み込みサポートを提供します。チェーン可能なルート修飾子やグループオプションを使用して個別ルートまたはルートグループ全体を保護できます。

クレデンシャルは`config/auth.js`の`auth.basic`配下でグローバルに設定するかルート定義で直接指定できます。

```javascript
// config/auth.js
export default {
  model: 'User',
  basic: {
    username: 'admin',
    password: 'password',
    realm: 'Restricted'
  }
};
```

<a name="ルートの保護"></a>

### ルートの保護

単一ルート、ルートグループ、またはチェーン可能なビルダーを使用して基本認証を適用します。

```javascript
// on a route group (uses configured default credentials)
Route.group({ basicAuth: true }, (admin) => {
  admin.get('/dashboard', 'admin.AdminController@index');
});

// on a route group with explicit credentials
Route.group({ basicAuth: { user: 'admin', pass: 'secret', realm: 'Admin Area' } }, (admin) => {
  admin.get('/settings', 'admin.SettingsController@index');
});

// group shorthand with credentials
Route.basicAuth('admin', 'secret', (admin) => {
  admin.get('/reports', 'admin.ReportController@index');
});

// as a chainable before routes (placeable in any order)
Route.basicAuth('admin', 'secret')
  .port(8080)
  .get('/metrics', 'app.MetricsController@index');

// on a single route modifier
Route.get('/secret', 'app.SecretController@show').basicAuth('admin', 'secret');

// with a custom validator callback (e.g. database query)
Route.basicAuth(async (user, pass) => {
  const account = await User.where('email', user).first();
  return account && await Hash.verify(pass, account.password);
}).get('/custom', 'app.CustomController@index');

// with custom validator and custom realm object
Route.basicAuth({
  validator: async (user, pass) => user === 'admin' && pass === 'secret',
  realm: 'Secure API'
}).get('/secure', 'app.SecureController@index');
```

クレデンシャルが存在しないか無効な場合フレームワークは即座に`401 Unauthorized`で応答し設定されたレルムを持つ`WWW-Authenticate`ヘッダーを設定します。クレデンシャルの比較にはタイミング攻撃を防ぐため定数時間比較が使用されます。

<a name="ShieldGuard"></a>

## ShieldGuard

標準のレート制限に加えてフレームワークは`ShieldGuard`というボット対策エンジンを備えています。

`ShieldGuard`はアクティブなセッションIDに紐付けられたタイムスタンプ、ソルト、hmac署名を含む暗号化チャレンジを生成することで動作します。クライアントは処理の続行を許可される前に指定された難易度に基づくプルーフオブワーク（pow）チャレンジを正常に解決する必要があります。ゲストチャレンジは未認証の訪問者にバインドされアカウントなしで収集されたチャレンジが認証済みセッションに対して再生されるのを防ぎます。

さらにこのエンジンはユーザーの環境と行動シグナル（タッチイベント、マウスの移動速度、ウィンドウの寸法）に関するクライアントサイドのテレメトリを収集します。クライアントサイドではshieldが状態を変更する`POST`フォーム送信を自動的にインターセプトしてチャレンジトークンを付与します。状態を変更しない`GET`送信やhtmlダイアログフォーム（`<form method="dialog">`）はバイパスされるため標準のブラウザクエリパラメータやダイアログの終了はネイティブに動作します。これらのシグナルはサーバーサイドで厳格に評価され「ボットスコア」が算出されます。スコアが許容しきい値を超えた場合リクエストは即座に拒否されます。

この透過的な防御レイヤーにより自動化されたスクリプトや悪意のあるボットがアプリケーションコントローラーに到達する前に確実に阻止されます。
