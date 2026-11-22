# ヘルパー

- [ヘルパー](#ヘルパー)
  - [はじめに](#はじめに)
  - [型チェック](#型チェック)
  - [値チェック](#値チェック)
  - [文字列と配列のユーティリティ](#文字列と配列のユーティリティ)
  - [乱数と日時](#乱数と日時)
  - [htmlとセキュリティ](#htmlとセキュリティ)
  - [配列ヘルパー](#配列ヘルパー)
  - [オブジェクトヘルパー](#オブジェクトヘルパー)
  - [ディープオブジェクトアクセス](#ディープオブジェクトアクセス)
  - [urlとアセットヘルパー](#urlとアセットヘルパー)
  - [ローカリゼーションとルビヘルパー](#ローカリゼーションとルビヘルパー)
  - [レスポンスヘルパー](#レスポンスヘルパー)
  - [ルーティングヘルパー](#ルーティングヘルパー)
  - [バリデーション](#バリデーション)
  - [データサニタイズ](#データサニタイズ)
  - [翻訳](#翻訳)
  - [ファサード](#ファサード)
    - [session](#session)
    - [auth](#auth)
    - [db](#db)
    - [job](#job)
    - [log](#log)
    - [app](#app)

<a name="はじめに"></a>

## はじめに

dframeworkはすべてのコントローラー、ミドルウェア、モデル、ビュー、ジョブでインポートなしに利用できるグローバルヘルパー関数のセットを提供します。これらのヘルパーは型チェック、データ操作、url生成、レスポンスショートカット、バリデーション、翻訳、コアサービスへのファサードアクセスをカバーします。

すべてのヘルパーはアプリケーションの初期化時にグローバルスコープへ登録されます。

<a name="型チェック"></a>

## 型チェック

| ヘルパー | シグネチャ | 説明 |
| -------------------- | ------------------ | ------------------------------------------------------------------------------------------------- |
| `isUndefined(value)` | `(any) => boolean` | 値が`undefined`の場合に`true`を返します |
| `isNull(value)` | `(any) => boolean` | 値が`null`の場合に`true`を返します |
| `isBoolean(value)` | `(any) => boolean` | 値が真偽値の場合に`true`を返します |
| `isNumber(value)` | `(any) => boolean` | 値が数値であり`NaN`でない場合に`true`を返します |
| `isInteger(value)` | `(any) => boolean` | 値が整数の場合に`true`を返します |
| `isString(value)` | `(any) => boolean` | 値が文字列の場合に`true`を返します |
| `isArray(value)` | `(any) => boolean` | 値が配列の場合に`true`を返します |
| `isObject(value)` | `(any) => boolean` | 値がプレーンオブジェクト（`null`でなく、配列でなく、コンストラクタが`Object`）の場合に`true`を返します |
| `isFunction(value)` | `(any) => boolean` | 値が関数の場合に`true`を返します |

```javascript
isString('hello');     // true
isNumber(42);          // true
isNumber(NaN);         // false
isObject({ a: 1 });    // true
isObject([1, 2]);      // false
isObject(null);        // false
```

<a name="値チェック"></a>

## 値チェック

**`isEmpty(value)`**

値が空と見なされる場合に`true`を返します。`undefined`、`null`、空文字列`''`、空配列`[]`、空のプレーンオブジェクト`{}`が空と判定されます。

```javascript
isEmpty('');           // true
isEmpty([]);           // true
isEmpty({});           // true
isEmpty(null);         // true
isEmpty(0);            // false
isEmpty('hello');      // false
isEmpty([1]);          // false
```

<a name="文字列と配列のユーティリティ"></a>

## 文字列と配列のユーティリティ

これらのヘルパーは文字列と配列の双方に対してポリモーフィックに動作します。

**`move(input, from, to)`**

文字列または配列内の要素をあるインデックスから別のインデックスへ移動します。元の値を変更せず新しい値を返します。

```javascript
move([1, 2, 3, 4], 0, 2);   // [2, 3, 1, 4]
move('abcd', 0, 2);          // "bcad"
```

**`remove(input, index)`**

文字列または配列から指定されたインデックスの要素を削除します。

```javascript
remove([1, 2, 3], 1);        // [1, 3]
remove('hello', 1);           // "hllo"
```

**`replace(input, index, newItem)`**

指定されたインデックスの要素を新しい値で置き換えます。

```javascript
replace([1, 2, 3], 1, 9);    // [1, 9, 3]
replace('hello', 0, 'H');    // "Hello"
```

**`limit(input, maxLength)`**

文字列または配列を指定された最大長に切り詰めます。

```javascript
limit('hello world', 5);     // "hello"
limit([1, 2, 3, 4, 5], 3);  // [1, 2, 3]
```

**`uniquify(input)`**

文字列（文字単位）または配列から重複する値を削除します。

```javascript
uniquify([1, 2, 2, 3, 3]);  // [1, 2, 3]
uniquify('aabbcc');           // "abc"
```

**`Inflector`**

`Inflector`エクスポートは`pluralize`、`snakeCase`、`pascalCase`、`inferTableName`を含む単語の語形変化およびケース変換ユーティリティを提供します。

```javascript
import { Inflector } from 'dframework';

Inflector.pluralize('person');       // "people"
Inflector.pluralize('user');         // "users"
Inflector.snakeCase('UserProfile');  // "user_profile"
Inflector.pascalCase('user_post');   // "UserPost"
Inflector.inferTableName('User');    // "users"
```

<a name="乱数と日時"></a>

## 乱数と日時

**`random(min, max)`**

`min`から`max`までのランダムな整数を生成します（両端を含みます）。

```javascript
random(1, 10);   // e.g. 7
random(0, 1);    // 0 or 1
```

**`now()`**

現在の日時をiso 8601形式の文字列として返します。

```javascript
now();   // "2026-06-05T02:30:00.000Z"
```

**`formatTime(seconds)`**

秒数を`mm:ss`表示形式にフォーマットします。

```javascript
formatTime(65);   // "01:05"
formatTime(360);  // "06:00"
```

**`formatDate(date, includeTime = true)`**

データベースおよびストレージの標準化のため、dateインスタンスまたは日付文字列をgmt 0の標準`YYYY-MM-DD`または`YYYY-MM-DD HH:mm:ss`形式にフォーマットします。無効な日付入力の場合はエラーをスローします。

```javascript
formatDate(new Date(), false);  // "2026-06-05"
formatDate(new Date());         // "2026-06-05 02:30:00"
```

**`localizeDate(date, includeTime = true, locale = null, timezone = null)`**

ハイフン区切りとスペース区切りの標準ロケール日付形式を使用して、ユーザーのタイムゾーンで日付をフォーマットします。

```javascript
localizeDate(new Date(), false);  // "06-05-2026"
localizeDate(new Date());         // "06-05-2026 02:30:00"
```

<a name="htmlとセキュリティ"></a>

## htmlとセキュリティ

**`escapeHtml(value)`**

xss攻撃を防ぐためにhtml特殊文字をエスケープします。`"`、`&`、`'`、`<`、`>`をエスケープします。`null`および`undefined`に対しては空文字列を返します。

```javascript
escapeHtml('<script>alert("xss")</script>');
// "&lt;script&gt;alert(&quot;xss&quot;)&lt;/script&gt;"

escapeHtml(null);   // ""
```

この関数は大きな文字列に対してシングルパスアルゴリズムを使用します。ビューエンジンの`{{ }}`構文はこの関数を自動的に呼び出します。

**`escapeJs(value)`**

javascript文字列リテラルおよびテンプレート文字列内で安全に使用できるように文字をエスケープします。`\`、`'`、`"`、`` ` ``、`$`、制御文字（改行、キャリッジリターン、タブ、nullバイト、バックスペース、フォームフィード）、`/`（`</script>`タグの脱出を防止するため）、unicode行区切り文字および段落区切り文字（`\u2028`および`\u2029`）をエスケープします。`null`および`undefined`に対しては空文字列を返します。

```javascript
escapeJs('hello "world", it\'s `cool` </script>');
// "hello \\\"world\\\", it\\\'s \\\`cool\\\` <\\/script>"

escapeJs(null);   // ""
```

**`emptyImage()`**

透明な1x1 gif画像のdata uriを返します。プレースホルダーのsrc属性として有用です。

```javascript
emptyImage();
// "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
```

<a name="配列ヘルパー"></a>

## 配列ヘルパー

**`arrayFirst(arr, callback)`**

コールバック関数を満たす最初の配列要素を返します。一致する要素がない場合は`undefined`を返します。

```javascript
arrayFirst([1, 2, 3, 4], n => n > 2);   // 3
```

**`arrayLast(arr, callback)`**

コールバック関数を満たす最後の配列要素を返します。

```javascript
arrayLast([1, 2, 3, 4], n => n < 4);    // 3
```

**`pluck(arr, key)`**

配列内の各オブジェクトから単一のプロパティを抽出します。

```javascript
pluck([{ name: 'tarou' }, { name: 'satou' }], 'name');
// ['tarou', 'satou']
```

**`merge(...args)`**

複数のオブジェクトまたは配列をディープマージします。配列は連結されます。オブジェクトは再帰的にマージされ、競合するキーについては後の値が前の値を上書きします。

```javascript
merge({ a: 1 }, { b: 2 });                    // { a: 1, b: 2 }
merge({ a: { x: 1 } }, { a: { y: 2 } });      // { a: { x: 1, y: 2 } }
merge([1, 2], [3, 4]);                        // [1, 2, 3, 4]
```

<a name="オブジェクトヘルパー"></a>

## オブジェクトヘルパー

**`old(key, fallback)`**

直前のリクエストからフラッシュされた入力値を取得します。バリデーション失敗後にフォームフィールドの値を再設定する際に有用です。

```javascript
old('email');                    // the previously submitted email value
old('name', 'default name');     // returns 'default name' if no old input exists
```

<a name="ディープオブジェクトアクセス"></a>

## ディープオブジェクトアクセス

これらのヘルパーはドット記法を使用して深くネストされたオブジェクトプロパティへの安全なアクセスを提供します。

**`get(obj, path, defaultValue)`**

ドット区切りのキーパスを使用してネストされた値を取得します。パスが存在しない場合は`defaultValue`を返します。

```javascript
const config = { database: { host: 'localhost', port: 3306 } };

get(config, 'database.host');              // 'localhost'
get(config, 'database.name', 'mydb');      // 'mydb'
get(config, 'missing.key');                // undefined
```

**`set(obj, path, value)`**

ドット区切りのキーパスを使用してネストされた値を設定します。必要に応じて中間オブジェクトを作成します。元のオブジェクトを変更して返します。

```javascript
const config = {};
set(config, 'database.host', 'localhost');
// { database: { host: 'localhost' } }
```

**`has(obj, path)`**

ドット区切りのパスがオブジェクト内に存在する場合に`true`を返します。

```javascript
has({ a: { b: 1 } }, 'a.b');    // true
has({ a: { b: 1 } }, 'a.c');    // false
```

**`forget(obj, path)`**

ドット区切りのパスによってネストされたプロパティを削除します。親が配列の場合は要素がスプライスされます。元のオブジェクトを変更して返します。

```javascript
const data = { user: { name: 'satou', age: 30 } };
forget(data, 'user.age');
// { user: { name: 'satou' } }
```

<a name="urlとアセットヘルパー"></a>

## urlとアセットヘルパー

**`asset(path)`**

publicディレクトリ内の静的アセットのurlを生成します。先頭のスラッシュを削除し単一の`/`を先頭に付加します。

```javascript
asset('css/app.css');     // '/css/app.css'
asset('/js/main.js');     // '/js/main.js'
```

**`storage(path)`**

storageディレクトリ内のファイルのurlを生成します。先頭に`/storage/`を付加します。

```javascript
storage('avatars/user-1.jpg');     // '/storage/avatars/user-1.jpg'
storage(null);                     // ''
```

**`url(path, params)`**

パラメータ置換を伴うurlを生成します。`:param`プレースホルダーをparamsオブジェクトの対応する値に置き換えます。値はurlエンコードされます。

```javascript
url('/users/:id', { id: 42 });
// '/users/42'

url('/posts/:slug/comments/:commentId', { slug: 'hello-world', commentId: 7 });
// '/posts/hello-world/comments/7'
```

<a name="ローカリゼーションとルビヘルパー"></a>

## ローカリゼーションとルビヘルパー

**`t(req, key, params?, fallback?)`**

`lang/{locale}/`のjsonファイルから指定されたキーのローカライズ文字列を取得します。`:name`変数置換およびインラインの`漢字[かんじ]`ふりがな解析をサポートします。

```javascript
await t(req, 'messages.greeting', { name: '太郎' });
// "こんにちは、太郎さん！"

await t(req, 'messages.welcome', { name: '山田' });
// "<ruby>東京<rt>とうきょう</rt></ruby>へようこそ、山田さん！"
```

**`ruby(baseOrText, reading?)`**

日本語ふりがなテキスト用のhtml `<ruby>base<rt>rt</rt></ruby>`注釈を生成します。2つの引数（漢字と読み）またはインラインの`漢字[かんじ]`注釈を含む単一の文字列を受け入れます。

```javascript
ruby('漢字', 'かんじ');
// "<ruby>漢字<rt>かんじ</rt></ruby>"

ruby('日本[にほん]語[ご]');
// "<ruby>日本<rt>にほん</rt></ruby><ruby>語<rt>ご</rt></ruby>"
```

<a name="レスポンスヘルパー"></a>

## レスポンスヘルパー

これらのヘルパーは現在のリクエストコンテキストのレスポンスオブジェクト上のメソッドを呼び出すためのショートカットです。

> [!NOTE]
> レスポンスヘルパーはアクティブなリクエストコンテキスト上で動作し、コントローラー、ミドルウェア、ルートハンドラー内でのみ使用できます。

| ヘルパー | 説明 |
| ------------------------------ | -------------------------------- |
| `render(view, data)`           | データとともにビューテンプレートをレンダリングします |
| `json(data)`                   | jsonレスポンスを送信します |
| `redirect(url, code, force)`   | 指定urlへリダイレクトします |
| `back(fallback, code, force)`  | 直前のurlへリダイレクトします |
| `abort(code, message)`         | httpエラーをスローします |
| `status(code)`                 | レスポンスステータスコードを設定します |
| `cookie(name, value, options)` | レスポンスcookieを設定します |
| `stream(writer)`               | ライターコールバックを使用してレスポンスをストリーミングします |

```javascript
export default class HomeController {
  index() {
    return render('home', { title: 'welcome' });
  }

  async store(req) {
    await Item.create(req.body);
    return redirect('/items');
  }

  async api(req) {
    const items = await Item.all();
    return json({ items });
  }
}
```

<a name="ルーティングヘルパー"></a>

## ルーティングヘルパー

**`route(name, params)`**

名前付きルートからurlを生成します。`:param`プレースホルダーをparamsオブジェクトの値に置き換えます。ドメインバインドされたルートの場合、ドメインパラメータの置換を伴うプロトコル相対絶対url（`//admin.example.com/dashboard`など）を返します。

```javascript
route('users.show', { id: 42 });
// '/users/42'

route('admin.dashboard');
// '//admin.example.com/dashboard'
```

このヘルパーを使用する前に`Route`ファサードが初期化されている必要があります。すべてのリクエストコンテキストで自動的に利用可能です。

**`routeIs(name, params?)`**

現在のリクエストのルートが指定された名前と一致するか判定します。`*`を使用したワイルドカードパターンをサポートします。オプションのparamsオブジェクトを受け取り、特定のルートパラメータに一致を限定できます。

```javascript
routeIs('dashboard');       // exact match
routeIs('admin.*');         // matches admin.users, admin.settings, etc.

// constrain to a specific document page
routeIs('docs.render', { slug: 'getting-started', page: 'installation' });
```

paramsが提供された場合、各キーは`req.params`の対応する値と厳密に一致（文字列比較）する必要があります。パラメータ化されたルートに対してどのアクティブナビゲーションリンクが有効かを判定するために、ビューテンプレート内で使用します。

<a name="バリデーション"></a>

## バリデーション

**`validate(dataOrReq, rules, messages)`**

ルールのセットに対してデータをバリデーションします。プレーンオブジェクトまたはリクエストオブジェクト（`body`と`files`が抽出されます）を受け入れます。

```javascript
const v = validate(req, {
  email: 'required|email',
  password: 'required|string|min:8',
  age: 'nullable|integer|min:18'
});

if (v.fails()) {
  return json({ errors: v.errors() });
}
```

バリデーション結果は以下のメソッドを提供します:

| メソッド | 戻り値の型 | 説明 |
| -------------- | ----------- | ------------------------------ |
| `passes()`     | `boolean`   | すべてのルールを通過した場合に`true` |
| `fails()`      | `boolean`   | いずれかのルールが失敗した場合に`true` |
| `errors()`     | `object`    | フィールド名をキーとするすべてのエラー |
| `first(field)` | `string \| null` | 指定フィールドの最初のエラーメッセージ |
| `all()`        | `object`    | `errors()`のエイリアス |

**利用可能なルール**

| ルール | 説明 |
| -------------- | ------------------------------------------------------------------- |
| `required`     | 値が空であってはなりません |
| `string`       | 値が文字列である必要があります |
| `number`       | 値が数値である必要があります |
| `integer`      | 値が整数である必要があります |
| `boolean`      | 値が真偽値形式である必要があります（`true`、`false`、`1`、`0`、`yes`、`no`） |
| `array`        | 値が配列である必要があります |
| `email`        | 値が有効なメールアドレスである必要があります |
| `min:value`    | 最小長（文字列や配列）または最小値（数値） |
| `max:value`    | 最大長（文字列や配列）または最大値（数値） |
| `in:a,b,c`     | 値がリストされた値のいずれかである必要があります |
| `not_in:a,b,c` | 値がリストされた値のいずれでもない必要があります |
| `nullable`     | 値が空の場合はそのフィールドのバリデーションルールをスキップします |

**ファイルバリデーションルール**

| ルール | 説明 |
| ---------------------- | ------------------------------------------------------ |
| `file`                 | ファイルが存在する必要があります |
| `file_required`        | ファイルのアップロードが必須です |
| `file_size:value`      | 最大ファイルサイズ（`kb`、`mb`、`gb`サフィックスをサポート） |
| `file_min_size:value`  | 最小ファイルサイズ |
| `file_mimes:type,type` | 許可されるmimeタイプまたはファイル拡張子 |
| `file_max_count:n`     | アップロード可能な最大ファイル数 |
| `file_min_count:n`     | アップロード可能な最小ファイル数 |

```javascript
const v = validate(req, {
  avatar: 'file_required|file_size:5mb|file_mimes:jpg,png,webp',
  documents: 'file|file_max_count:10|file_size:20mb'
});
```

**カスタムエラーメッセージ**

`field.rule`をキーとするmessagesオブジェクトを渡すことで、デフォルトのエラーメッセージをオーバーライドできます。

```javascript
const v = validate(req, {
  email: 'required|email'
}, {
  'email.required': 'please enter your email address',
  'email.email': 'the email format is invalid'
});
```

<a name="データサニタイズ"></a>

## データサニタイズ

**`sanitizeBody(data, fields)`**

フィールドの型定義に基づいて生の入力データをサニタイズおよびキャストします。処理前にフォームデータが正しい型にキャストされていることを保証するのに有用です。

```javascript
const cleaned = sanitizeBody(req.body, [
  { name: 'age', type: 'integer' },
  { name: 'score', type: 'number' },
  { name: 'active', type: 'boolean' },
  { name: 'tags', type: 'array' },
  { name: 'metadata', type: 'json' }
]);
```

| 型 | 動作 |
| --------- | -------------------------------------------------------------------------------- |
| `number`  | 文字列を浮動小数点数にパースし、数値はそのまま通します |
| `integer` | 文字列を整数にパースし、整数はそのまま通します |
| `boolean` | `'true'`、`'1'`、`'yes'`を`true`に、`'false'`、`'0'`、`'no'`を`false`に変換します |
| `array`   | 文字列からjson配列をパースするか、カンマ区切り文字列を分割します |
| `json`    | json文字列をオブジェクトにパースします |
| default   | 値を変更せずにそのまま通します |

値が`null`または`undefined`のフィールドは結果から除外されます。

<a name="翻訳"></a>

## 翻訳

**`t(key, fallback)` または `t(req, key, fallback)`**

現在のロケールの翻訳文字列を取得します。リクエストオブジェクトは任意であり、リクエストコンテキストから自動解決されます。詳細については[ローカリゼーション](../other/localization.md)のドキュメントを参照してください。

```javascript
// auto resolves request from context
const welcome = await t('common.welcome');
const missing = await t('common.unknown', 'fallback text');

// or pass request explicitly
const explicit = await t(req, 'common.welcome');
```

<a name="ファサード"></a>

## ファサード

dframeworkはコアサービス用のグローバルファサードを提供します。これらのファサードはグローバルスコープで利用可能であり、現在のアプリケーションインスタンスのサービスへ解決されます。

<a name="session"></a>

### session

`Session`ファサードは現在のリクエストのセッションストアへのアクセスを提供します。

| メソッド | 説明 |
| ---------------------------------- | ------------------------------------------------------------------ |
| `Session.get(key, fallback)`       | セッションから値を取得します |
| `Session.set(key, value)`          | セッションに値を保存します |
| `Session.all()`                    | すべてのセッションデータをオブジェクトとして取得します |
| `Session.has(key)`                 | セッションにキーが存在するか判定します |
| `Session.forget(key)`              | 値を削除します（キーが渡されない場合はセッション全体を破棄します） |
| `Session.flush()`                  | アクティブなセッションを破棄し、保存されているすべてのデータを削除します |
| `Session.flash(key, value)`        | 次のリクエストでのみ有効な値を保存します |
| `Session.permanent(data)`          | 長期生存cookie（10年間）でデータを保存します |
| `Session.id(newId)`                | アクティブなセッションidを返却または設定します |
| `Session.regenerate(preserveData)` | セッションidを再生成します（デフォルトでデータを保持します） |

```javascript
await Session.set('theme', 'dark');
const theme = await Session.get('theme', 'light');
const hasTheme = await Session.has('theme');
await Session.flash('status', 'saved successfully');
await Session.regenerate();
const sid = Session.id();
```

<a name="auth"></a>

### auth

`Auth`ファサードは認証済みユーザー、ガード、認証アクションへのアクセスを提供します。

| メソッド | 説明 |
| ----------------------------------- | ----------------------------------------------------- |
| `Auth.user()`                       | 認証済みユーザーオブジェクトまたは`null`を返します |
| `Auth.check()`                      | ユーザーが認証されている場合に`true`を返します |
| `Auth.id()`                         | 認証済みユーザーidまたは`null`を返します |
| `Auth.guard(name)`                  | ガード固有の認証メソッド（`user`、`check`）を返します |
| `Auth.login(user, data, permanent)` | デフォルトガードでユーザーをログインさせます |
| `Auth.logout()`                     | デフォルトガードからログアウトします |
| `Auth.flush()`                      | すべてのガード状態とセッションデータをクリアします |

```javascript
if (Auth.check()) {
  const user = Auth.user();
  const userId = Auth.id();
}

// using guards
if (Auth.guard('admin').check()) {
  const admin = Auth.guard('admin').user();
}

await Auth.login(user, { role: 'admin' });
await Auth.guard('admin').login(adminUser);

// log out of default guard only
await Auth.logout();

// log out of admin guard only
await Auth.guard('admin').logout();

// flush all guards and sessions
await Auth.flush();
```

`Auth.login()`および`Auth.guard().login()`はオプションの第3引数を受け入れます。`true`に設定すると、セッションはブラウザセッションcookieの代わりに長期生存cookieを使用します。

<a name="db"></a>

### db

`DB`ファサードはデータベースインスタンスへの直接アクセスを提供します。すべての呼び出しをアプリケーションのデータベース接続に委譲します。

```javascript
const users = await DB.table('users').where('active', true).get();
const count = await DB.table('orders').count();
await DB.query('SELECT * FROM users WHERE id = ?', [1]);
```

<a name="job"></a>

### job

`Job`ファサードはワーカープールへバックグラウンドジョブをディスパッチします。詳細については[ジョブとキュー](../asynchronous/jobs-and-queues.md)のドキュメントを参照してください。

```javascript
Job.dispatch('SendEmailJob', { to: 'user@example.com', template: 'welcome' });
```

<a name="log"></a>

### log

`Log`ファサードはアプリケーションロガーへのグローバルアクセスを提供します。詳細については[ロギングとエラー](../other/logging-and-errors.md)のドキュメントを参照してください。

```javascript
Log.debug('processing started');
Log.info('user registered:', user.email);
Log.warn('rate limit approaching');
Log.error('payment failed:', err);
Log.table(rows, 'query results');
```

<a name="app"></a>

### app

`app(key?)`ヘルパーは現在のアプリケーションインスタンスまたは登録されたサービスへのアクセスを提供します。利用可能な場合はアクティブなリクエストコンテキストから解決し、それ以外はグローバルアプリケーションインスタンスにフォールバックします。

```javascript
// retrieve the active application instance
const application = app();

// access specific services or properties directly
const db = app('db');
const logger = app('logger');
const env = app('env');
```
