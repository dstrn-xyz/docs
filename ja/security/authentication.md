# 認証

- [認証](#認証)
  - [はじめに](#はじめに)
  - [設定](#設定)
  - [ユーザーの認証](#ユーザーの認証)
    - [ログイン](#ログイン)
    - [ログアウト](#ログアウト)
    - [認証状態の確認](#認証状態の確認)
    - [認証済みユーザーの取得](#認証済みユーザーの取得)
    - [ユーザーIDの取得](#ユーザーIDの取得)
  - [マルチガード認証](#マルチガード認証)
    - [ガードの定義](#ガードの定義)
    - [ガードの使用](#ガードの使用)
    - [すべてのガードのフラッシュ](#すべてのガードのフラッシュ)
  - [ハッシュ化](#ハッシュ化)
    - [標準ハッシュ化](#標準ハッシュ化)
    - [決定論的高速ハッシュ化](#決定論的高速ハッシュ化)
  - [基本認証](#基本認証)
    - [ルート修飾子](#ルート修飾子)
    - [チェーン可能なビルダー](#チェーン可能なビルダー)
    - [動的バリデーター](#動的バリデーター)
    - [ルートグループ](#ルートグループ)

<a name="はじめに"></a>

## はじめに

dframeworkでは認証の実装が極めて簡単です。グローバルな`Auth`ファサードはアプリケーション全体でユーザーセッションと認証状態を管理するためのシンプルで統一されたapiを提供します。

<a name="設定"></a>

## 設定

認証設定は`config/auth.js`に記述します。デフォルトでは単一モデル認証設定が`User`を参照しています。

```javascript
// config/auth.js
export default {
  model: 'User'
};
```

セッションクッキー名は`config/session.js`内の`cookie`または`SESSION_COOKIE`環境変数（デフォルトは`sid`）でカスタマイズできます。

```javascript
// config/session.js
export default {
  cookie: Env.value('SESSION_COOKIE', 'sid'),
};
```

<a name="ユーザーの認証"></a>

## ユーザーの認証

<a name="ログイン"></a>

### ログイン

ユーザーをアプリケーションにログインさせるには`Auth`ファサードの`login`メソッドを使用します。このメソッドはユーザーモデルインスタンスを受け取ります。第2引数として追加のセッションデータを渡すことも可能であり第3引数にセッションを永続的（長期生存）にするかどうかを示すブール値を渡すこともできます。

> [!NOTE]
> ユーザーが`Auth.login()`を介してログインするとそのIDが安全に保存されセッション固定化攻撃を緩和するためにセッション識別子が自動的に再生成されます。

```javascript
import { Hash } from 'dframework';
import User from '../models/User.js';

export async function authenticate(req, res) {
  const user = await User.findByEmail('tarou@example.com');

  if (user && await Hash.verify('secret', user.password)) {
    // log the user in and set a long lived session cookie
    await Auth.login(user, { role: 'admin' }, true);
    
    return redirect('/dashboard');
  }

  return back('/login').withErrors({ email: 'invalid credentials' });
}
```

<a name="ログアウト"></a>

### ログアウト

デフォルトガードからユーザーをログアウトさせるには`logout`メソッドを使用します。これにより他のアクティブなガードセッションを保持したままデフォルトガードのセッションキーがクリアされます。

```javascript
await Auth.logout();
```

<a name="認証状態の確認"></a>

### 認証状態の確認

現在のリクエストが認証されているかどうかを判定するには`check`メソッドを使用します。ユーザーがログインしている場合は`true`を返します。

```javascript
if (Auth.check()) {
  // the user is logged in
}
```

<a name="認証済みユーザーの取得"></a>

### 認証済みユーザーの取得

`Auth`ファサードの`user`メソッドを介して認証済みユーザーにアクセスできます。これはアクティブなユーザーモデルインスタンスを返しユーザーが未認証の場合は`null`を返します。

```javascript
const user = Auth.user();

if (user) {
  Log.info(`welcome back, ${user.name}`);
}
```

<a name="ユーザーIDの取得"></a>

### ユーザーIDの取得

プロパティを手動で参照することなく認証済みユーザーのIDを迅速に取得するには`id`メソッドを使用します。

```javascript
const userId = Auth.id();
```

<a name="マルチガード認証"></a>

## マルチガード認証

アプリケーションで複数の異なるエンティティタイプ（例えば一般ユーザーと管理者）に対して独立した認証が必要な場合は`config/auth.js`でガードを定義できます。

<a name="ガードの定義"></a>

### ガードの定義

ガードは`config/auth.js`の`guards`オブジェクト配下に登録します。各ガードはモデルとオプションのカスタムセッションキー（デフォルトは`${guardName}Id`）を指定します。

```javascript
// config/auth.js
export default {
  default: 'web',
  guards: {
    web: {
      model: 'User',
      sessionKey: 'userId'
    },
    admin: {
      model: 'Admin',
      sessionKey: 'adminId'
    }
  }
};
```

<a name="ガードの使用"></a>

### ガードの使用

特定のガードで認証アクションを実行するには`Auth`ファサードの`guard`メソッドを使用します。

```javascript
// check if an admin is logged in
if (Auth.guard('admin').check()) {
  const admin = Auth.guard('admin').user();
  const adminId = Auth.guard('admin').id();
}

// log in an admin (preserves active user sessions on other guards)
await Auth.guard('admin').login(adminUser);

// log out admin only
await Auth.guard('admin').logout();
```

すべてのガード読み取り（`user()`、`check()`、`id()`）はリクエスト中完全に同期的です。

<a name="すべてのガードのフラッシュ"></a>

### すべてのガードのフラッシュ

アクティブなすべてのガードログインを完全にクリアしセッションデータを1回の呼び出しで消去するには`flush`を使用します。

```javascript
await Auth.flush();
```

<a name="ハッシュ化"></a>

## ハッシュ化

dframeworkは安全なパスワードハッシュ化にbcryptを使用する`Hash`ファサードを提供します。

<a name="標準ハッシュ化"></a>

### 標準ハッシュ化

パスワードをハッシュ化するには`make`メソッドを使用します。安全なソルトを自動的に生成し12ラウンドのbcryptハッシュを適用します。

```javascript
import { Hash } from 'dframework';

const hashedPassword = await Hash.make('my-password');
```

平文パスワードをハッシュと照合して検証するには`verify`メソッドを使用します。

```javascript
if (await Hash.verify('plain-text', hashedPassword)) {
  // passwords match
}
```

<a name="決定論的高速ハッシュ化"></a>

### 決定論的高速ハッシュ化

エントロピーの高いトークン（apiキーやパーソナルアクセストークンなど）をデータベースでの完全一致検索が可能な方法で安全にハッシュ化したい場合があります。`fast`メソッドはサーバーサイドの`APP_KEY`をペッパーとして使用した決定論的sha256ハッシュを提供します。

```javascript
// generating a token and saving its fast hash
const token = Hash.random(40);
const hashedToken = Hash.fast(token);

// querying the database for the exact hashed token
const record = await TokenModel.where('token', Hash.fast('provided-token')).first();
```

<a name="基本認証"></a>

## 基本認証

dframeworkは`Route`ファサードを介してhttp基本認証（basic authentication）の組み込みサポートを直接提供します。チェーン可能なルートビルダー、ルート修飾子、グループオプション、または簡略記法グループを使用して個別ルートまたはルートグループ全体を保護できます。

クレデンシャルは`config/auth.js`の`auth.basic`配下でグローバルに設定するかルート定義で直接指定できます。

<a name="ルート修飾子"></a>

### ルート修飾子

ルート定義に直接`.basicAuth()`をチェーンします。

```javascript
// using credentials configured in config/auth.js
Route.get('/metrics', 'app.MetricsController@show').basicAuth();

// with explicit credentials and custom realm
Route.get('/metrics', 'app.MetricsController@show').basicAuth('admin', 'secret123', 'internal area');
```

<a name="チェーン可能なビルダー"></a>

### チェーン可能なビルダー

`.domain()`、`.port()`、`.middleware()`と並んで任意の順序でルート定義の前に`.basicAuth()`を配置できます。

```javascript
Route.basicAuth('admin', 'secret123', 'internal area')
  .port(8080)
  .get('/metrics', 'app.MetricsController@show');
```

<a name="動的バリデーター"></a>

### 動的バリデーター

データベースやカスタムストレージに対してクレデンシャルを検証するために非同期のバリデーターコールバックを渡します。

```javascript
Route.basicAuth(async (username, password) => {
  const admin = await Admin.where('username', username).first();
  if (!admin) return false;
  return await Hash.verify(password, admin.password);
}).get('/dashboard', 'app.DashboardController@index');

// with a custom realm
Route.basicAuth({
  validator: async (username, password) => {
    const admin = await Admin.where('username', username).first();
    if (!admin) return false;
    return await Hash.verify(password, admin.password);
  },
  realm: 'Admin Area'
}).get('/dashboard', 'app.DashboardController@index');
```

<a name="ルートグループ"></a>

### ルートグループ

グループオプションまたはグループ簡略記法を使用して複数のルートを基本認証で保護します。

```javascript
// group options using config defaults
Route.group({ basicAuth: true }, (admin) => {
  admin.get('/dashboard', 'admin.AdminController@index');
});

// group shorthand with config defaults and zero parameter callback
Route.basicAuth(() => {
  Route.get('/status', 'admin.StatusController@show');
});

// group shorthand with explicit credentials
Route.basicAuth('admin', 'secret', (admin) => {
  admin.get('/settings', 'admin.SettingsController@index');
});

// group shorthand with custom validator
Route.basicAuth(async (user, pass) => user === 'admin' && pass === 'secret', (admin) => {
  admin.get('/panel', 'admin.PanelController@index');
});
```
