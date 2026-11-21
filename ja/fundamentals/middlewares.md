# ミドルウェア

- [ミドルウェア](#ミドルウェア)
  - [はじめに](#はじめに)
  - [ミドルウェアの定義](#ミドルウェアの定義)
  - [next関数](#next関数)
  - [チェーンの停止](#チェーンの停止)
  - [ミドルウェアの登録](#ミドルウェアの登録)
  - [ミドルウェアパラメータ](#ミドルウェアパラメータ)
  - [ホットリロード](#ホットリロード)
  - [レート制限](#レート制限)

<a name="はじめに"></a>

## はじめに

ミドルウェアはアプリケーションに届くhttpリクエストを検査およびフィルタリングするための便利な仕組みを提供します。例えば認証ミドルウェアはユーザーがログインしているかを検証します。ログインしていない場合ミドルウェアはログイン画面にリダイレクトします。認証されている場合ミドルウェアはリクエストがアプリケーションのより深い処理へ進むことを許可します。

<a name="ミドルウェアの定義"></a>

## ミドルウェアの定義

すべてのカスタムミドルウェアは`middlewares`ディレクトリに配置します。ミドルウェアは非同期メソッドを持つクラスとして定義します。各メソッドは独立したミドルウェアハンドラーとして機能します。

```javascript
// middlewares/AuthMiddleware.js
export default class AuthMiddleware {
  async requireAuth(req, next) {
    if (!Auth.check()) {
      return redirect('/login', true);
    }
    return next();
  }

  async requireGuest(req, next) {
    if (Auth.check()) {
      return redirect('/home', true);
    }
    return next();
  }
}
```

cliを使用してミドルウェアのスタブを生成できます。

```bash
dstrn make:middleware AuthMiddleware
```

<a name="next関数"></a>

## next関数

すべてのミドルウェアメソッドは`req`オブジェクトと`next`クロージャを受け取ります。実行チェーン内の次のミドルウェアに制御を渡すには`next()`を呼び出す必要があります。`next()`を呼び出さない場合リクエストはそのミドルウェアで停止しフレームワークはレスポンスが処理されたとみなします。

### コントローラとビューへのデータ受け渡し

`next()`にオブジェクトペイロードを渡すか`.with()`をチェーンすることで後続のコントローラとビューの両方にデータを渡すことができます:

```javascript
export default class AuthMiddleware {
  async requireAuth(req, next) {
    if (!Auth.check()) {
      return redirect('/login', true);
    }

    const devices = await Device.where({ user_id: Auth.user().id, revoked: false });

    // passes data to req.devices and merges it into all view templates
    return next({ devices });
  }
}
```

チェーン可能な`.with()`構文を使用することもできます:

```javascript
return next().with({ devices, currentDevice });
```

`next(data)`または`next.with(data)`経由で渡されたデータはすべて自動的に以下の処理が行われます:
- 後続のコントローラ用に`req`オブジェクトにマージ（`req.devices`としてアクセス可能）
- ビューテンプレート用にビューコンテキストにマージ（`.d`テンプレート内で`devices`として直接アクセス可能）

レスポンスオブジェクトへのアクセスが必要なミドルウェアもあります。その場合は第2引数として`res`を第3引数として`next`を宣言できます。

```javascript
export default class CustomMiddleware {
  async intercept(req, res, next) {
    res.set('X-Custom-Header', 'dstrn');
    return next();
  }
}
```

<a name="チェーンの停止"></a>

## チェーンの停止

リクエストを停止してコントローラに到達しないようにするには`next()`を呼び出す代わりに`redirect()`、`abort()`、`json()`などのレスポンスヘルパーを返します。

```javascript
// middlewares/AdminMiddleware.js
export default class AdminMiddleware {
  async requireAdmin(req, next) {
    if (!Auth.check()) return redirect('/login');
    if (Auth.user().power < 5) return abort(403, 'unauthorized access');
    
    return next();
  }
}
```

> [!IMPORTANT]
> ミドルウェアランナーはレスポンスの状態を厳格に監視します。ミドルウェアがヘッダーを送信したか書き込み可能なストリームを終了した場合誤って`next()`が呼び出されたとしてもチェーンは即座に中断されます。

<a name="ミドルウェアの登録"></a>

## ミドルウェアの登録

ミドルウェアはグローバルには登録されません。`ClassName@method`という文字列構文を使用してルートに直接アタッチします。ルーターはこれらの文字列を自動的に解決します。

単一のルートにミドルウェアを適用できます。

```javascript
Route.middleware('AuthMiddleware@requireAuth').get('/profile', 'app.UserProfileController@show');
```

ルートのグループにミドルウェアを適用できます。

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.get('/dashboard', 'app.IndexController@dashboard');
  auth.get('/library', 'app.IndexController@library');
});
```

配列を指定して複数のミドルウェアをチェーンできます。

```javascript
Route.group({ middleware: ['AuthMiddleware@requireAuth', 'AdminMiddleware@requireAdmin'] }, (admin) => {
  admin.get('/users', 'admin.AdminController@list');
});
```

最後の項目を除くすべての項目がミドルウェアとして扱われるインライン配列を使用してルートを定義することもできます。

```javascript
Route.post('/device/approve', ['AuthMiddleware@requireAuth', 'auth.AuthController@approvePost']);
```

<a name="ミドルウェアパラメータ"></a>

## ミドルウェアパラメータ

dframeworkはミドルウェア文字列構文を介して直接パラメータを渡すことをサポートしていません。ミドルウェアに動的な設定が必要な場合はクラス上に個別のメソッドを定義するか`req`オブジェクトを介してデータを渡す必要があります。

```javascript
export default class DeviceMiddleware {
  async verifyDevice(req, next) {
    const devices = await Device.where({ user_id: Auth.id(), revoked: false });
    
    // attach data to the request for controllers and views
    return next({ devices });
  }
}
```

<a name="ホットリロード"></a>

## ホットリロード

`local`環境ではフレームワークは`middlewares`ディレクトリを監視します。ミドルウェアファイルを変更するとランナーはそのキャッシュを自動的に無効化してクラスを再読み込みします。これによりサーバーを再起動することなくミドルウェアのロジックが即座にホットリロードされます。

<a name="レート制限"></a>

## レート制限

組み込みの`RateLimiter`はスライディングウィンドウアルゴリズムを使用します。設定可能な時間枠内でキーごとのリクエスト数を追跡し期限切れのエントリを自動的にクリーンアップして標準的なレート制限レスポンスヘッダーを設定します。

```javascript
import { RateLimiter } from 'dframework';

const limiter = new RateLimiter({
  windowMs: 60 * 1000, // 1 minute
  max: 60,             // 60 requests per window
  message: 'too many requests, please slow down',
});

Route.middleware(limiter).group({ prefix: '/api' }, (Route) => {
  Route.post('/messages', 'MessageController@store');
  Route.post('/uploads',  'UploadController@store');
});
```

制限を超過した場合レスポンスには`429`ステータスとともに`Retry-After`、`X-RateLimit-Limit`、`X-RateLimit-Remaining`ヘッダーが含まれます。

| オプション | デフォルト | 説明 |
| --- | --- | --- |
| `windowMs` | `60000` | ミリ秒単位の時間枠 |
| `max` | `60` | ウィンドウあたりの最大リクエスト数 |
| `message` | `'too many requests'` | 制限超過時に返されるメッセージ |
| `keyGenerator` | クライアントip | `(req) => string`、カスタムグループキー |
| `trustedProxies` | `[]` | `x-forwarded-for`解決のための信頼できるプロキシip |
| `maxEntries` | `100000` | lru退去前の最大追跡キー数 |
| `headers` | `false` | 制限未超過のリクエストでもクォータヘッダーを送信 |

カスタムの`keyGenerator`はユーザー単位での制限に役立ちます:

```javascript
const limiter = new RateLimiter({
  windowMs: 60 * 1000,
  max: 20,
  keyGenerator: (req) => Auth.id(),
});
```
