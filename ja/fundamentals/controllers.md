# コントローラ

- [コントローラ](#コントローラ)
  - [はじめに](#はじめに)
  - [コントローラの記述](#コントローラの記述)
  - [ディレクトリ構造と命名規則](#ディレクトリ構造と命名規則)
  - [省略可能な引数](#省略可能な引数)
  - [レスポンスの返却](#レスポンスの返却)
  - [リクエストへのアクセス](#リクエストへのアクセス)
    - [ルートパラメータ](#ルートパラメータ)
    - [クエリ文字列](#クエリ文字列)
    - [リクエストボディ](#リクエストボディ)
    - [ファイルアップロード](#ファイルアップロード)
    - [クッキー](#クッキー)
    - [ページネーション](#ページネーション)
  - [バリデーション](#バリデーション)
  - [モデルとサービスの使用](#モデルとサービスの使用)
  - [プライベートメソッド](#プライベートメソッド)
  - [ホットリロード](#ホットリロード)

<a name="はじめに"></a>

## はじめに

コントローラはリクエスト処理ロジックを集約されたクラスに整理します。dframeworkはルートが一致した際に適切なコントローラクラスを自動的に検出してインスタンス化します。開発者が手動でコントローラをインスタンス化する必要はありません。

<a name="コントローラの記述"></a>

## コントローラの記述

コントローラはデフォルトエクスポートを持つクラスです。各メソッドはルートアクションに対応します。

```javascript
// controllers/app/IndexController.js
export default class IndexController {
  index() {
    return render('auth.index');
  }

  register() {
    return render('auth.register');
  }
}
```

cliを使用してコントローラのスタブを生成できます。

```bash
dstrn make:controller app/IndexController
```

<a name="ディレクトリ構造と命名規則"></a>

## ディレクトリ構造と命名規則

コントローラは`controllers`ディレクトリ内に配置されサブディレクトリに整理できます。サブディレクトリとクラス名はルート定義文字列に直接反映されます。

`controllers/app/HomeController.js`にあるコントローラは`app.HomeController@method`として参照されます。`controllers/auth/AuthController.js`にあるコントローラは`auth.AuthController@method`として参照されます。ルート直下の`controllers/LocaleController.js`にあるコントローラは`LocaleController@method`として参照されます。

```javascript
Route.get('/home', 'app.HomeController@home');
Route.post('/register', 'auth.AuthController@register');
Route.get('/locale/:locale', 'LocaleController@set');
```

<a name="省略可能な引数"></a>

## 省略可能な引数

コントローラメソッドは引数として`req`と`res`を受け取ります。どちらも省略可能です。メソッドが実際に使用するもののみを宣言します。メソッドがリクエストをまったく必要としない場合は完全に省略できます。

```javascript
export default class IndexController {
  index() {
    return render('auth.index');
  }

  async user(req) {
    const targetUser = await User.find(req.params.id);
    return render('app.user.profile', { targetUser });
  }
}
```

<a name="レスポンスの返却"></a>

## レスポンスの返却

コントローラはフレームワークが公開するグローバルヘルパーを使用してレスポンスを返します。レスポンスオブジェクトのメソッドを直接呼び出すことはありません。

```javascript
export default class PreferencesController {
  async index() {
    const preferences = await UserPreference.getAllForUser(Auth.user().id);
    return json({ success: true, preferences });
  }

  async update(req) {
    await UserPreference.setForUser(Auth.user().id, req.params.key, req.body.value);
    return json({ success: true });
  }

  destroy() {
    return status(500).json({ success: false, message: 'failed to reset preference' });
  }
}
```

利用可能なレスポンスヘルパーは`render()`、`json()`、`redirect()`、`back()`、`abort()`、`status()`、`cookie()`、`stream()`です。コントローラメソッドは常にこれらのヘルパーのいずれかを返す必要があります。レスポンスヘルパーのみがステータスコード、コンテンツタイプ、ビューのコンパイルを管理するため素のモデルインスタンスやプレーンオブジェクトを直接返さないでください。

> [!NOTE]
> 手動で`status()`を設定するよりも通常は`abort(statusCode, message)`ヘルパーを使用することが推奨されます。`abort()`はコンテキストを認識します。リクエストがajax経由で行われた場合はjsonエラーペイロードを自動的に返します。標準的なブラウザリクエストの場合は適切なhtmlエラーページ（404や500など）をレンダリングします。

```javascript
export default class UserController {
  async show(req) {
    const user = await User.find(req.params.id);
    if (!user) {
      // aborts with a 404, returning json for ajax requests or a 404 view for standard requests
      return abort(404, 'user not found');
    }
    return render('app.user.profile', { user });
  }
}
```

> [!IMPORTANT]
> `redirect(url, code = 302, force = false)`ヘルパーはオープンリダイレクト脆弱性に対して厳格に保護されています。クロスオリジンやプロトコル相対urlへのリダイレクトを拒否し代わりにエラーをスローします（ローカル開発環境を除く）。さらにspaも認識します。リクエストがdSPAルーターからのものである場合標準の302ヘッダーの代わりにjson命令を返します。`force`ブール値を`true`に設定するとクライアント側でspaルーターに強制的なハードリロードを実行させます。

```javascript
export default class AuthController {
  async logout(req) {
    await Auth.logout();
    // safely redirects to the home page, instructing the spa router to handle it
    return redirect('/home');
  }
}
```

<a name="リクエストへのアクセス"></a>

## リクエストへのアクセス

<a name="ルートパラメータ"></a>

### ルートパラメータ

ルート定義で`:`がプレフィックスとして付けられたurlセグメントは抽出され`req.params`で利用できます。

```javascript
export default class IndexController {
  async user(req) {
    const targetUser = await User.find(req.params.id);
    return render('app.user.profile', { targetUser });
  }
}
```

<a name="クエリ文字列"></a>

### クエリ文字列

クエリ文字列の値はプレーンオブジェクトとして`req.query`で利用できます。フレームワークは初回アクセス時にクエリ文字列を遅延解析します。

```javascript
export default class SearchController {
  async index(req) {
    const { q, sort } = req.query;
    const results = await Track.where('title', q).get();
    return json({ results });
  }
}
```

<a name="リクエストボディ"></a>

### リクエストボディ

解析されたリクエストボディは`req.body`で利用できます。フレームワークはコントローラメソッドが呼び出される前に更新系リクエストのjsonおよびフォームエンコードされたボディを自動的に解析します。

```javascript
export default class PreferencesController {
  async update(req) {
    const { key } = req.params;
    const { value } = req.body;
    await UserPreference.setForUser(Auth.user().id, key, value);
    return json({ success: true, key, value });
  }
}
```

<a name="ファイルアップロード"></a>

### ファイルアップロード

リクエストが`multipart/form-data`として送信された場合フレームワークはファイルを解析して`req.files`で利用可能にします。`req.files`の各キーはファイルオブジェクトの配列です。また`req.file(key)`を使用して最初のファイルを直接取得したり`req.hasFile(key)`を使用してファイルがアップロードされたかを確認したりすることもできます。

```javascript
export default class UserProfileController {
  async updateProfileImage(req) {
    if (!req.hasFile('profileImage')) {
      return status(422).json({ error: 'no image provided' });
    }
    const image = req.file('profileImage');
    const path = await Storage.disk('public').put(`user_assets/${Auth.user().id}/${image.originalFilename}`, image);
    return json({ success: true, path });
  }
}
```

<a name="クッキー"></a>

### クッキー

受信したクッキーはプレーンオブジェクトとして`req.cookies`で利用できます。フレームワークは初回アクセス時にcookieヘッダーを遅延解析します。

```javascript
export default class LocaleController {
  set(req) {
    const preferred = req.cookies.locale;
    return json({ preferred });
  }
}
```

<a name="ページネーション"></a>

### ページネーション

dframeworkはクエリ文字列パラメータからアクティブなページ番号を自動的に読み取ります。任意のモデルクエリで`.paginate()`を呼び出すとリクエストコンテキストから現在のページを解決し`Paginator`インスタンスを返します:

```javascript
export default class AdminController {
  async list() {
    const users = await User.where('active', 1).paginate(20);
    return json(users);
  }
}
```

`json(users)`経由で返されるとペジネーターはレコードとページネーションメタデータの両方を含むクリーンなレスポンスにシリアライズされます。

`Paginator`インスタンスはsnake_caseとcamelCaseの両方でのプロパティアクセス、ループ内での直接反復、およびヘルパーメソッドをサポートしています:

- `data` / `items`: 現在のページのモデルインスタンスの配列
- `total`: 全ページの総レコード数
- `per_page` / `perPage`: 1ページあたりのレコード数
- `current_page` / `currentPage`: 現在のページ番号
- `last_page` / `lastPage`: 総ページ数
- `next_page_url` / `nextPageUrl`: クエリパラメータが保持された次ページのurl（存在しない場合は`null`）
- `prev_page_url` / `prevPageUrl`: クエリパラメータが保持された前ページのurl（存在しない場合は`null`）
- `first_page_url` / `firstPageUrl`: 最初のページのurl
- `last_page_url` / `lastPageUrl`: 最後のページのurl
- `from`: 現在のページの最初のレコードの1始まりのインデックス
- `to`: 現在のページの最後のレコードの1始まりのインデックス
- `hasPages()`: 複数ページが存在する場合にtrueを返します
- `hasMorePages()`: 最後のページでない場合にtrueを返します
- `onFirstPage()`: 1ページ目の場合にtrueを返します
- `onLastPage()`: 最後のページの場合にtrueを返します
- `url(page)`: クエリパラメータを保持して指定ページのurlを構築します
- `setPath(path)`: 生成されるページネーションリンクのベースurlパスを設定します（チェーン可能）
- `setPageName(name)`: ページナビゲーションに使用されるクエリパラメータ名を設定します（チェーン可能）
- `appends(params)`: 生成されるurlにカスタムクエリパラメータを追加します

同一ページ上で複数の異なるコレクションをページネーションするにはカスタムページパラメータ名を指定します:

```javascript
const users = await User.paginate(10, 'user_page');
const logs = await AuditLog.paginate(10, 'log_page');
```

<a name="バリデーション"></a>

## バリデーション

グローバルな`validate()`ヘルパーは任意のコントローラメソッド内で利用可能です。ファイルアップロードを検証する場合はリクエストボディオブジェクトまたは完全な`req`オブジェクトのいずれかを渡します。利用可能なルールの完全な一覧については[バリデーションドキュメント](../security/validation.md)を参照してください。

```javascript
export default class UserProfileController {
  updateUsername(req) {
    const validation = validate(req.body, {
      username: 'required|string|min:3|max:20'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }

    // proceed with update
  }

  updateProfileImage(req) {
    const validation = validate(req, {
      profileImage: 'file_required|file_size:8mb|file_max_count:1'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }

    // proceed with upload
  }
}
```

<a name="モデルとサービスの使用"></a>

## モデルとサービスの使用

コントローラは通常ファイルの先頭でモデルとサービスをインポートします。フレームワークは`local`環境でコントローラをホットリロードするためこれらのインポートはファイル変更ごとに再評価されます。

```javascript
import Track from '../../models/Track.js';
import User from '../../models/User.js';

export default class HomeController {
  async home(req) {
    const leaderboard = await User.orderBy('current_rank', 'ASC').limit(5).get();
    const popularTracks = await Track.where('is_popular', 1).limit(5).get();

    return render('app.home.index', { leaderboard, popularTracks });
  }
}
```

<a name="プライベートメソッド"></a>

## プライベートメソッド

コントローラはプレーンなクラスであるためルートハンドラーと並んで非asyncのヘルパーメソッドを定義できます。フレームワークはルート定義に一致するメソッドのみを呼び出します。クラス上のその他のメソッドは開発者のロジック専用のプライベートなものとなります。

```javascript
export default class PreferencesController {
  async update(req) {
    const validation = this.validatePreference(req.params.key, req.body.value);
    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }
    await UserPreference.setForUser(Auth.user().id, req.params.key, req.body.value);
    return json({ success: true });
  }

  validatePreference(key, value) {
    return validate({ value }, { value: this.getValidationRules(key) });
  }

  getValidationRules(key) {
    const rules = {
      'audio.normalize_volume': 'boolean',
      'display.fps_counter': 'boolean',
    };
    return rules[key] || 'required';
  }
}
```

<a name="ホットリロード"></a>

## ホットリロード

`local`環境ではフレームワークはファイル変更を検知するために`controllers`ディレクトリを監視します。コントローラファイルを保存するとフレームワークはサーバーを再起動することなく次のリクエスト時にそれを自動的に再インポートします。この動作を有効にするために特別な設定を行う必要はありません。
