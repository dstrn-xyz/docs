# モデル

- [モデル](#モデル)
  - [はじめに](#はじめに)
  - [モデルの定義](#モデルの定義)
    - [テーブル名](#テーブル名)
    - [主キー](#主キー)
    - [属性のキャスト](#属性のキャスト)
    - [キャストの確認](#キャストの確認)
  - [モデルの取得](#モデルの取得)
    - [メソッド一覧](#メソッド一覧)
    - [マジックファインダー](#マジックファインダー)
    - [ページネーション](#ページネーション)
    - [最新レコードの取得](#最新レコードの取得)
    - [クエリの複製](#クエリの複製)
  - [挿入と更新](#挿入と更新)
    - [複数代入](#複数代入)
    - [複数代入ヘルパー](#複数代入ヘルパー)
    - [ライフサイクルフック](#ライフサイクルフック)
    - [レコードの検索または作成](#レコードの検索または作成)
    - [インクリメントとデクリメント](#インクリメントとデクリメント)
  - [モデルの削除](#モデルの削除)
  - [リレーション](#リレーション)
    - [1対1](#1対1)
    - [1対多](#1対多)
    - [多対1](#多対1)
    - [多対多](#多対多)
    - [Eager Loading](#eager-loading)
    - [Lazy Eager Loading](#lazy-eager-loading)
  - [シリアライズ](#シリアライズ)
    - [属性の非表示](#属性の非表示)
    - [属性の表示](#属性の表示)
  - [ミューテーション](#ミューテーション)

<a name="はじめに"></a>

## はじめに

dframeworkにはデータベースと対話するためのアクティブレコード実装が含まれています。各データベーステーブルにはそのテーブルとのやりとりに使用される対応するモデルがあります。モデルを使用することで洗練された流暢なインターフェースによりテーブル内のデータを照会したりテーブルに新しいレコードを挿入したりできます。

<a name="モデルの定義"></a>

## モデルの定義

モデルを作成するにはフレームワークが提供する基本の`Model`クラスを拡張します。

```javascript
import { Model } from 'dframework';

export default class User extends Model {
}
```

<a name="テーブル名"></a>

### テーブル名

デフォルトではクラス名を小文字の複数形にしたものがテーブル名として使用されます。例えば`User`モデルは`users`テーブルが存在することを前提とします。テーブルがこの命名規則に従っていない場合は静的な`table`プロパティをオーバーライドすることでカスタムテーブル名を指定できます。

```javascript
export default class User extends Model {
  static table = 'system_users';
}
```

<a name="主キー"></a>

### 主キー

フレームワークはデータベーススキーマを検査して結果をキャッシュすることによりテーブルの主キーを自動的に判別します。この動作をオーバーライドしたい場合は静的な`primaryKey`プロパティを定義できます。

```javascript
export default class User extends Model {
  static primaryKey = 'uuid';
  static keyType = 'uuid';
}
```

`static keyType = 'uuid'`が設定されている場合モデルは`create()`および`save()`の実行時にUUID主キーの生成と解決を自動的に処理します。

主キーカラム名にかかわらずモデルインスタンスの主キー値を取得するには`model.getKey()`を呼び出します。

スキーマに定義されている場合は複合主キーも自動的にサポートされます。

<a name="属性のキャスト"></a>

### 属性のキャスト

属性のキャストはデータベースカラムとJavaScriptデータ型との間の自動変換を提供します。モデルに静的な`casts`オブジェクトを定義して特定のカラムのキャストを設定します。

```javascript
export default class User extends Model {
  static casts = {
    metadata: 'json',
    tags: 'array',
    options: 'object',
    is_active: 'boolean',
    login_count: 'int',
    score: 'float',
    joined_at: 'datetime',
    display_name: 'string'
  };
}
```

#### サポートされているキャストタイプ

| キャストタイプ | 説明 |
| :------------------------------ | :--------------------------------------------------------------------------------------------------------------------------------------- |
| `json`、`array`、`object` | ハイドレーション時に文字列化されたJSONをJavaScriptオブジェクトまたは配列にパースし、`save()`および`create()`時にオブジェクトまたは配列を文字列化します。 |
| `boolean`、`bool` | 値を真偽値（`true`または`false`）にキャストします（`'1'`、`'0'`、`'true'`、`'false'`、`1`、`0`を処理）。 |
| `int`、`integer` | `parseInt()`を使用して値を整数にキャストします。 |
| `float`、`double`、`real` | `parseFloat()`を使用して値を浮動小数点数にキャストします。 |
| `date`、`datetime`、`timestamp` | ハイドレーション時に値を`Date`オブジェクトにキャストし保存時にUTCの標準SQLタイムスタンプ形式にフォーマットします。postgresql、mysql、sqlite全体でデータベースタイムスタンプはすべてUTCに標準化されます。 |
| `string` | 値をプリミティブ文字列にキャストします。 |

#### JSON属性の直接変更

属性が`json`、`array`、`object`にキャストされている場合メモリ内でオブジェクトを直接変更し`save()`で変更を永続化できます。

```javascript
const user = await User.find(1);

user.metadata.theme = 'dark';
user.metadata.notifications = true;

await user.save();
```

サブクラスはプロトタイプチェーン上の親モデルクラスから静的な`casts`を自動的に継承およびマージします。

#### 自動JSONキャスト

モデルは明示的なキャストを必要とせずにJSONデータを自動的に検出およびシリアライズします。プレーンなJavaScriptオブジェクトまたは配列を含む属性は`save()`、`create()`、クエリ更新時にJSON文字列へとシリアライズされます。同様にJSONとしてフォーマットされたデータベース文字列（`{}`または`[]`で始まり終わる文字列）はハイドレーション時に自動的にJavaScriptオブジェクトへパースされます。未加工のJSON形式文字列をパースせずに文字列のまま保持するには明示的に`'string'`キャストを宣言してください。

<a name="キャストの確認"></a>

#### キャストの確認

モデルで解決されたキャスト定義を確認するにはモデルクラスまたはインスタンスで`getCasts()`を呼び出します。このメソッドはプロトタイプ継承チェーンをたどり親のすべてのキャストをマージします:

```javascript
const casts = User.getCasts();
```

<a name="モデルの取得"></a>

## モデルの取得

モデルは流暢なクエリビルダーのすべてのメソッドをプロキシするため結果を取得する前に制約をチェーンできます。`all`メソッドはモデルのテーブルからすべてのレコードを取得します。

```javascript
const users = await User.all();
```

特定のレコードを主キーで取得するには`find`メソッドを使用します。

```javascript
const user = await User.find(1);
```

<a name="メソッド一覧"></a>

### メソッド一覧

クエリを実行するすべてのメソッドは`Promise`を返します。戻り値列にはnullや空配列のセマンティクスを含めPromiseが解決する内容を示します。

| メソッド | 引数 | 戻り値 |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `Model.all()` | なし | `Promise<Model[]>`（ハイドレーションされたインスタンス、行がない場合は`[]`） |
| `Model.query()` | なし | `ModelQueryBuilder`（新しいクエリビルダーインスタンス） |
| `Model.find(id)` | 主キーの値、複合主キーの場合は`{ pk1, pk2 }` | `Promise<Model\|null>` |
| `Model.first()` | なし | `Promise<Model\|null>`（フィルタリングなしの先頭レコード。条件指定にはfirstWhereまたはwhere().first()を使用） |
| `Model.firstWhere(col, op?, val?)` | カラム名、演算子または値、オプションの値 | `Promise<Model\|null>` |
| `Model.latest(count?, column?)` | `null`または数値、オプションのカラム名 | countが`null`: `Promise<Model\|null>`、数値count: `Promise<Model[]>` |
| `Model.where(column, op?, val?)` | カラム/演算子/値、またはオブジェクト | `ModelQueryBuilder`（チェーン可能、thenable、非同期反復可能） |
| `Model.orderBy() / groupBy() / limit() / offset() / distinct()` | 各メソッドのチェーン値 | `ModelQueryBuilder`（チェーン可能） |
| `Model.with(relations)` | ドット記法のリレーション名 | `ModelQueryBuilder`（チェーン可能） |
| `Model.paginate(perPage, pageName?)` | 1ページあたりの行数（デフォルト10）、オプションのページパラメータ名 | `Promise<Paginator>` |
| `Model.firstOrCreate(attributes, values?)` | 属性オブジェクト、オプションの追加値 | `Promise<Model>`（既存または新規作成されたインスタンス） |
| `Model.updateOrCreate(attributes, values?)` | 属性オブジェクト、オプションの追加値 | `Promise<Model>`（更新または新規作成されたインスタンス） |
| `Model.create(data)` | カラムと値のオブジェクト | `Promise<Model\|null>`（主キーが割り当てられたハイドレーション済みインスタンス。ライフサイクルフックによって中止された場合はnull。SELECTによる再ロードはされません） |
| `Model.isFillable(key)` | 属性名文字列 | `boolean`（属性が複数代入可能な場合はtrue） |
| `Model.filterAttributes(data)` | カラムと値のオブジェクト | `object`（複数代入可能な属性のみを含む浅いコピー） |
| `Model.getCasts()` | なし | `object`（継承チェーン全体で解決されたマージ済みキャスト定義） |
| `builder.clone()` | なし | `ModelQueryBuilder`（リレーションと制約を保持した独立したコピー） |
| `instance.save(newData?)` | 保存前にマージするオプションのオブジェクト | `Promise<boolean>`（成功時はtrue、フックによって中止された場合はfalse） |
| `instance.update(data)` | カラムと値のオブジェクト | `Promise<boolean>`（インプレースで変更しUPDATEを実行。中止された場合はfalse） |
| `instance.touch()` | なし | `Promise<boolean>`（データベース内のupdated_atタイムスタンプを更新） |
| `instance.increment(column, amount?, extra?)` | カラム名、オプションの加算量（デフォルト1）、オプションの追加カラムオブジェクト | `Promise<object>`（インスタンス属性をインプレースで変更しインクリメントUPDATEを実行） |
| `instance.decrement(column, amount?, extra?)` | カラム名、オプションの減算量（デフォルト1）、オプションの追加カラムオブジェクト | `Promise<object>`（インスタンス属性をインプレースで変更しデクリメントUPDATEを実行） |
| `instance.delete()` | なし | `Promise<boolean>`（主キーにより行を削除。フックによって中止された場合はfalse） |
| `instance.hash(field)` | インスタンス上の属性名 | `Promise<void>`（インスタンスをインプレースで変更） |
| `instance.load(relations)` | リレーション名 | `Promise<Model>`（リレーションが設定された同じインスタンス） |
| `instance.toJSON()` | なし | 非表示キーが削除されリレーションがシリアライズされたプレーンオブジェクト |
| `Model.getHidden()` | なし | プロトタイプチェーン全体で継承されたすべての非表示属性の配列 |
| `Model.getVisible()` | なし | モデルに定義されたすべての表示属性の配列またはnull |

`where`チェーン可能ビルダーは流暢なクエリビルダーと同じ制約、クロージャグループ化、条件分岐（`when`、`unless`）、サブクエリ（`whereExists`、`whereIn`、`selectSub`）、単一エンティティ検索（`find`、`first`）、集計メソッドを公開します（`where`、`whereIn`、`whereNull`、`whereBetween`、`whereColumn`、`whereHashed`、`whereExists`、`when`、`unless`、`selectSub`、`join`、`leftJoin`、`rightJoin`、`crossJoin`、`joinRaw`、`count`、`sum`、`avg`、`min`、`max`など）。またthenable（`await builder`）および非同期反復可能（`for await (const m of builder)`）でもあります。

<a name="マジックファインダー"></a>

### マジックファインダー

フレームワークは特定のカラムでレコードを取得するための動的なマジックメソッドを提供します。`findBy`プレフィックスにキャメルケースでカラム名を追加するだけです。

```javascript
const user = await User.findByEmail('test@example.com');
```

<a name="ページネーション"></a>

### ページネーション

レコードをページ分割するには`paginate`メソッドを使用します。現在のリクエストコンテキストからアクティブなページクエリ文字列パラメータを自動的に読み取りlimitおよびoffset制約を構築して`Paginator`インスタンスを返します。

```javascript
const results = await User.where('status', 'active').paginate(15);
```

2番目の引数としてカスタムのページクエリパラメータ名を指定することもできます:

```javascript
const users = await User.paginate(15, 'user_page');
```

ページネーターをビューに渡す前にベースURL、パラメータ名、または追加のクエリパラメータを流暢に設定するには:

```javascript
results.setPath('/search').setPageName('p').appends({ q: 'query' });
```

<a name="最新レコードの取得"></a>

### 最新レコードの取得

`latest`メソッドは指定カラムの降順で行をソートし単一のモデルまたは配列を返します。

```javascript
const newestPost = await Post.latest();
const recentPosts = await Post.latest(10);
const recentlyPublished = await Post.latest(10, 'published_at');
```

引数なしで`latest()`を呼び出すと最新の単一インスタンス（またはnull）を返します。数値の`count`を渡すとその件数のモデルインスタンスを配列として返します。カラムのデフォルトは`created_at`ですが2番目の引数でオーバーライドできます。

<a name="クエリの複製"></a>

### クエリの複製

`ModelQueryBuilder`で`clone()`を呼び出すとクエリチェーンの独立したコピーが作成されます。複製されたインスタンスはベーステーブルクエリ、アクティブな制約句、Eager Loading定義（`with`）、リレーションメタデータを保持します:

```javascript
const base = User.where('status', 'active').with('profile');

const admins = await base.clone().where('role', 'admin').get();
const editors = await base.clone().where('role', 'editor').get();
```

<a name="挿入と更新"></a>

## 挿入と更新

新しいレコードを挿入するには新しいモデルインスタンスをインスタンス化し属性を設定して`save`メソッドを呼び出します。

```javascript
const user = new User();
user.name = 'tarou';
user.email = 'tarou@example.com';
await user.save();
```

<a name="複数代入"></a>

### 複数代入

dframeworkはLaravelの規約に一致する`fillable`および`guarded`静的モデルプロパティを介して複数代入保護を提供します。デフォルトでは`fillable`も`guarded`も定義されていない場合（または`guarded = []`の場合）すべての属性が複数代入可能です。許可されるカラム名の配列として`fillable`（許可リスト）を指定するか保護されるカラム名の配列として`guarded`（拒否リスト、またはすべての属性を保護する`['*']`）を指定することで代入可能な属性を制限できます。

プロパティの直接代入またはコンストラクタ経由で事前定義された主キーを持つ新しいモデルを作成し`save()`を呼び出すと挿入が試みられキーの衝突時にエラーがスローされます。

```javascript
export default class User extends Model {
  static fillable = ['name', 'email'];
}
```

`create()`または`update()`を使用する場合`fillable`で許可され`guarded`で保護されていない属性のみが割り当てられます:

```javascript
const user = await User.create({
  name: 'tarou',
  email: 'tarou@example.com',
  is_admin: 1
});
```

<a name="複数代入ヘルパー"></a>

#### 複数代入ヘルパー

特定の属性キーが複数代入可能かどうかを確認したり任意の入力データをフィルタリングしたりするには`isFillable`および`filterAttributes`静的メソッドを使用できます:

```javascript
User.isFillable('name');
User.isFillable('is_admin');

const payload = { name: 'tarou', email: 'tarou@example.com', is_admin: 1 };
const safeAttributes = User.filterAttributes(payload);
```

<a name="ライフサイクルフック"></a>

### ライフサイクルフック

モデルはレコード変更時に実行されるライフサイクルフックをサポートしています:

- `creating(instance)` / `created(instance)`: 挿入の前後で発行
- `updating(instance)` / `updated(instance)`: 更新の前後で発行
- `saving(instance)` / `saved(instance)`: 挿入および更新の両方の前後で発行
- `deleting(instance)` / `deleted(instance)`: レコード削除の前後で発行

フックはクラスメソッドとして直接宣言することも`Model.creating(fn)`、`Model.created(fn)`、`Model.updating(fn)`、`Model.updated(fn)`、`Model.saving(fn)`、`Model.saved(fn)`、`Model.deleting(fn)`、`Model.deleted(fn)`を介して静的に登録することもできます。

事前フック（`creating`、`updating`、`saving`、`deleting`）から`false`を返すと操作が中止されてデータベースへの書き込みが防止され`save()`、`update()`、または`delete()`は`false`に解決されます。正常に完了するとこれらのメソッドは`true`に解決されます。

```javascript
export default class User extends Model {
  creating() {
    if (!this.email) return false;
  }

  created() {
  }
}

User.creating((user) => {
  if (!user.email) return false;
});
```

モデルを更新するにはプロパティを変更して`save`を呼び出すか`update`メソッドを直接使用します。

```javascript
const user = await User.find(1);
await user.update({ status: 'active' });
```

他の属性を変更せずにモデルの`updated_at`タイムスタンプのみを更新する必要がある場合は`touch`メソッドを使用します:

```javascript
const user = await User.find(1);
await user.touch();
```

<a name="レコードの検索または作成"></a>

### レコードの検索または作成

`firstOrCreate`メソッドは指定されたカラムと値のペアを使用してレコードの特定を試みます。モデルが見つからない場合1番目の引数の属性と2番目の引数の任意のオプション属性を使用してレコードが挿入されます。

```javascript
const user = await User.firstOrCreate(
  { email: 'tarou@example.com' },
  { name: 'tarou' }
);
```

`updateOrCreate`メソッドも利用できます。更新が適用された既存のインスタンスまたは一致するものがなかった場合は新しいインスタンスを返します。

<a name="インクリメントとデクリメント"></a>

### インクリメントとデクリメント

既存のモデルインスタンスのカラムをインクリメントまたはデクリメントするには`increment`および`decrement`メソッドを使用します。これらのメソッドはメモリ内のインスタンス属性を更新しモデルの主キーを対象とするアトミックなデータベース更新文を即座に実行します。

```javascript
const user = await User.find(1);

await user.increment('total_listens');

await user.increment('credits', 10, { updated_at: '2026-09-14' });

await user.decrement('credits', 5);
```

<a name="モデルの削除"></a>

## モデルの削除

モデルを削除するにはモデルインスタンスで`delete`メソッドを呼び出します。

```javascript
const user = await User.find(1);
await user.delete();
```

<a name="リレーション"></a>

## リレーション

モデルは他のモデルへのリレーションを定義でき関連するデータを流暢に走査および照会できます。すべてのリレーションメソッドは`ModelQueryBuilder`（チェーン可能、thenable、await可能）を返します。

<a name="1対1"></a>

### 1対1

1対1のリレーションは`hasOne`メソッドを使用して定義します。関連するモデルクラスと外部キー名が必要です。

```javascript
import Profile from './Profile.js';

export default class User extends Model {
  profile() {
    return this.hasOne(Profile, 'user_id');
  }
}
```

<a name="1対多"></a>

### 1対多

1対多のリレーションは`hasMany`メソッドを使用して定義します。

```javascript
import Post from './Post.js';

export default class User extends Model {
  posts() {
    return this.hasMany(Post, 'user_id');
  }
}
```

<a name="多対1"></a>

### 多対1

`hasOne`または`hasMany`リレーションの逆は`belongsTo`メソッドを使用して定義します。

```javascript
import User from './User.js';

export default class Post extends Model {
  user() {
    return this.belongsTo(User, 'user_id');
  }
}
```

<a name="多対多"></a>

### 多対多

多対多のリレーションは`belongsToMany`メソッドを使用して定義します。規約によりフレームワークは両方のモデル名をアルファベット順にソートしたスネークケース（例: `role_user`）で中間テーブル名を自動的に推測し外部ピボットキーを`${singular}_id`（例: `user_id`および`role_id`）として導出します。

```javascript
import Role from './Role.js';

export default class User extends Model {
  roles() {
    return this.belongsToMany(Role);
  }
}
```

スキーマでカスタム名が使用されている場合は明示的な中間テーブルとキー引数を渡すこともできます:

```javascript
return this.belongsToMany(Role, 'user_roles', 'usr_id', 'role_id', 'id', 'id');
```

#### ピボットデータの取得とwithPivot

`belongsToMany`リレーションを照会またはEager Loadingすると中間テーブルの属性はプライマリモデル自体の属性を汚染することなく各関連インスタンスの`model.pivot`に自動的に添付されます。

中間テーブルから追加のカラムを読み込むには`withPivot`をチェーンします:

```javascript
export default class User extends Model {
  roles() {
    return this.belongsToMany(Role).withPivot('expires_at', 'level');
  }
}

const user = await User.with('roles').first();
console.log(user.roles[0].pivot.level);
```

#### 中間レコードの関連付け、解除、同期

リレーションクエリビルダーには中間レコードを管理するための便利なヘルパーが用意されています:

```javascript
const user = await User.find(1);

await user.roles().attach(2, { level: 'admin' });

await user.roles().attach([3, 4]);

await user.roles().attach({
  5: { level: 'manager' },
  6: { level: 'viewer' }
});

await user.roles().detach(2);
await user.roles().detach([3, 4]);

await user.roles().detach();

const diff = await user.roles().sync({
  1: { level: 'admin' },
  5: { level: 'editor' }
});
```

リレーションが定義されるとメソッドを呼び出して照会でき`ModelQueryBuilder`が返されます。

```javascript
const activePosts = await user.posts().where('status', 'active').get();
```

`hasMany`/`belongsTo`/`hasOne`リレーションビルダーをawaitするとハイドレーションされたモデルが直接返されるため（`hasMany`の場合は配列、`hasOne`/`belongsTo`の場合は単一モデルまたはnull）次のように記述することもできます:

```javascript
const profile = await user.profile();
const posts = await user.posts();
```

関数として呼び出さずにプロパティを直接awaitすることも可能です:

```javascript
const posts = await user.posts;
```

#### インメモリキャッシュと遅延フェッチ

フレームワークはEager Loadingされたリレーションを自動的にキャッシュしawait時に遅延ロードを透過的に処理します:

- `with('posts')`または`load('posts')`でリレーションがEager Loadingされていた場合`await user.posts`（または`await user.posts()`、または並行する`Promise.all([user.profile(), user.posts()])`）はデータベースクエリなしでメモリ内で即座に解決されます。
- リレーションがEager Loadingされていなかった場合リレーションをawaitすると要求に応じて単一のデータベースクエリが実行されます。
- モデルにnullの外部キーまたはローカルキーがある場合（未割り当てのリレーションなど）制約のないリレーションクエリはデータベースを照会せずに即座に`null`または`[]`に解決されます。

リレーションプロパティは呼び出し可能なプロキシであるため（インメモリコレクションアクセスと`user.posts().where('status', 'active')`のような流暢なクエリチェーンの両方を可能にします）`Array.isArray(user.posts)`は`false`を返します。データアクセスを最適化するために手動で`Array.isArray`の分岐を行う必要はありません。事前ロードされているかどうかにかかわらず単に`await user.posts`または`await user.posts()`を使用するだけでハイドレーション済みのコレクションを取得できます。

#### リレーションプロパティの代入と変更

リレーションプロパティはゲッターとセッターの両方をサポートしています。モデルインスタンス上でEager Loadingされたコレクションまたは関連モデルを直接代入できます:

```javascript
user.posts = [customPostA, customPostB];
user.profile = new Profile({ bio: 'updated bio' });
```

リレーションプロパティを変更するとリレーション状態が更新され`user.toJSON()`によるシリアライズ時にも保持されます。

<a name="eager-loading"></a>

### Eager Loading

リレーションにプロパティとしてアクセスするとフレームワークは事前ロードされたリレーションデータを読み取ります。N+1クエリ問題を防止するには親モデルを取得する際に`with`メソッドを使用してリレーションをEager Loadingします。

```javascript
const users = await User.with('profile', 'posts').limit(10).get();

const user = await User.with('profile').find(1);

for (const user of users) {
  console.log(user.posts);
}
```

ドット記法を使用してネストされたリレーションをEager Loadingできます。

```javascript
const users = await User.with('posts.comments').get();
```

関連モデルをフィルタリングまたはソートするためのクエリビルダーのクロージャを渡してEager Loadingリレーションを制約することもできます。クロージャ内で適用された順序は厳密に保持されます:

```javascript
const users = await User.with('posts', (query) => {
  query.where('published', true).orderBy('views', 'desc');
}).get();

const authors = await Author.with({
  articles: (query) => query.where('status', 'live').orderBy('created_at', 'desc'),
  comments: (query) => query.orderBy('id', 'asc')
}).get();
```

> [!TIP]
> ローカル開発中デバッグバーは不足しているEager Loadingによる繰り返しのクエリを自動的に検出しアクティブな呼び出し元に対する推奨の`with()`構文を含むインタラクティブなモーダルを表示します。

<a name="lazy-eager-loading"></a>

### Lazy Eager Loading

すでにモデルインスタンスを取得しており事後的にリレーションをEager Loadingする必要がある場合は`load`メソッドを使用します。

```javascript
const user = await User.find(1);
const sameUser = await user.load('posts', 'profile');
```

<a name="シリアライズ"></a>

## シリアライズ

モデルをJSON文字列にキャストするかルートから返すとフレームワークは`toJSON`メソッドを使用して自動的に変換します。このメソッドは`hidden`リストを適用した上ですべての属性とEager Loadingされたリレーションを含むプレーンなオブジェクトを返します。

<a name="属性の非表示"></a>

### 属性の非表示

パスワードや機密トークンなどの特定の属性をシリアライズされた出力から隠したい場合があります。これらの属性を除外するにはモデルに静的な`hidden`配列を定義します。

```javascript
export default class User extends Model {
  static hidden = ['password', 'secret', 'api_key'];
}
```

> [!NOTE]
> デフォルトではフレームワークは`toJSON()`でモデルをシリアライズする際に`password`、`token`、`secret`、`api_key`、`remember_token`を自動的に非表示にします。追加の非表示属性を定義するサブクラスはこれらのデフォルトを自動的に継承します。非表示属性は`toJSON()`のシリアライズ（APIレスポンスなど）にのみ適用されコード内でのプロパティへの直接アクセス（認証時の`user.password`の読み取りなど）を制限することはありません。

<a name="属性の表示"></a>

### 属性の表示

あるいは静的な`visible`プロパティを使用してモデルをシリアライズする際に含めるべき属性の許可リストを定義することもできます。`visible`が定義されている場合指定された属性、リレーション、ピボットキーのみが`toJSON()`でシリアライズされ`hidden`は無視されます。

```javascript
export default class User extends Model {
  static visible = ['id', 'name', 'email', 'posts', 'pivot'];
}
```

<a name="ミューテーション"></a>

## ミューテーション

モデルはプロパティアクセスを基盤となる属性ストアに動的にプロキシします。通常のJavaScriptオブジェクトであるかのようにモデルインスタンスと対話できます。

クエリビルダーの自動ハッシュ設定で処理されない機密フィールドを手動で暗号化する必要がある場合はモデルインスタンスで`hash`メソッドを使用できます。

```javascript
const user = await User.find(1);
user.custom_secret = 'plain-text';
await user.hash('custom_secret');
await user.save();
```
