# クエリビルダー

- [クエリビルダー](#クエリビルダー)
  - [はじめに](#はじめに)
  - [メソッド一覧](#メソッド一覧)
  - [結果の取得](#結果の取得)
    - [すべての行の取得](#すべての行の取得)
    - [単一行の取得](#単一行の取得)
    - [最新レコードの取得](#最新レコードの取得)
    - [単一カラム値の取得（pluck）](#単一カラム値の取得pluck)
    - [集計（count、sum、avg、min、max）](#集計countsumavgminmax)
  - [select句](#select句)
    - [サブクエリselect](#サブクエリselect)
  - [where句](#where句)
    - [基本のwhere句](#基本のwhere句)
    - [論理グループ化（whereクロージャ）](#論理グループ化whereクロージャ)
    - [or文](#or文)
    - [その他のwhere句](#その他のwhere句)
    - [whereInでのサブクエリ](#whereinでのサブクエリ)
    - [whereExistsとwhereNotExists](#whereexistsとwherenotexists)
    - [カラム間比較](#カラム間比較)
  - [条件分岐句（whenとunless）](#条件分岐句whenとunless)
  - [join（結合）](#join結合)
    - [内部結合（inner join）](#内部結合inner-join)
    - [左外部結合 / 右外部結合](#左外部結合--右外部結合)
    - [高度な結合句](#高度な結合句)
    - [クロス結合（cross join）](#クロス結合cross-join)
    - [生クエリによる結合（raw joins）](#生クエリによる結合raw-joins)
  - [ソート、グループ化、制限](#ソートグループ化制限)
  - [挿入](#挿入)
    - [一括挿入](#一括挿入)
  - [更新](#更新)
    - [インクリメントとデクリメント](#インクリメントとデクリメント)
  - [削除](#削除)
  - [クエリの複製](#クエリの複製)
  - [自動ハッシュ化](#自動ハッシュ化)

<a name="はじめに"></a>

## はじめに

dframeworkはデータベースクエリを構築するための流暢でチェーン可能なクエリビルダーを提供します。プリペアドステートメントのみに依存することでSQLインジェクション攻撃から保護します。バインディングを手動でクリーンアップまたはサニタイズする必要はありません。

グローバルに利用可能な`DB`ファサードの`table`メソッドを呼び出してクエリビルダーチェーンを開始します。

```javascript
const builder = DB.table('users');
// builder: TableQuery. chainable, thenable, and async iterable.
```

<a name="メソッド一覧"></a>

## メソッド一覧

クエリビルダーは制約メソッド（ビルダーを返す）をチェーンして構築されターミナルメソッド（Promiseを返す）で終了します。以下の戻り値列には実際の解決値を示します。

### チェーン可能な制約メソッド

これらのメソッドはそれぞれチェーン用に同じ`TableQuery`インスタンスを返します。

| メソッド | 引数 |
| -------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `from(table)`                                                  | 対象テーブル名またはエイリアス（ビルダーのソーステーブルを更新） |
| `select(columns)`                                              | カラム名（現在のselectリストを置き換え） |
| `selectRaw(expression, bindings?)`                             | 生のSQLフラグメントと任意のバインディング |
| `selectSub(query, as)`                                         | エイリアス付きスカラサブクエリ式 |
| `distinct(columns?)`                                           | distinctを指定（オプションでselectリストを置き換え） |
| `where(column, operator?, value?)`                             | カラム名/値、`{ column: value }`オブジェクト、またはクロージャ |
| `whereNot(条件)`                                               | `where`の否定形（クロージャをサポート） |
| `orWhere(条件)`                                                | `where`のor形（クロージャをサポート） |
| `orWhereNot(条件)`                                             | `whereNot`のor形（クロージャをサポート） |
| `whereIn(column, values)`                                      | カラム名と値の配列、またはサブクエリクロージャ |
| `whereNotIn(column, values)`                                   | `whereIn`の否定形（サブクエリクロージャをサポート） |
| `orWhereIn(条件) / orWhereNotIn(条件)`                         | IN句のor形式 |
| `whereExists(callback) / whereNotExists(callback)`           | exists / not existsサブクエリチェック |
| `orWhereExists(callback) / orWhereNotExists(callback)`       | existsチェックのor形式 |
| `whereNull(column) / whereNotNull(column)`                   | nullチェック |
| `orWhereNull(column) / orWhereNotNull(column)`               | nullチェックのor形式 |
| `whereBetween(column, [min, max]) / whereNotBetween(条件)`    | 範囲チェック（厳密に2つの値が必要） |
| `orWhereBetween(条件) / orWhereNotBetween(条件)`               | 範囲チェックのor形式 |
| `whereColumn(column, operator?, otherColumn)`                  | 同じ行内の2つのカラムを比較 |
| `orWhereColumn(column, operator?, otherColumn)`                | カラム比較のor形式 |
| `whereRaw(sql, bindings?)`                                     | 生のSQL where条件（任意のバインディング付き） |
| `orWhereRaw(sql, bindings?)`                                   | 生のSQL or where条件（任意のバインディング付き） |
| `whereHashed(column, operator?, value) / orWhereHashed(条件)` | 値の高速ハッシュ値との比較 |
| `when(value, callback, defaultCallback?)`                      | 値がtruthyの場合にコールバックを適用 |
| `unless(value, callback, defaultCallback?)`                    | 値がfalsyの場合にコールバックを適用 |
| `join(table, first, operator?, second)`                        | 別のテーブルとの内部結合（joinクロージャをサポート） |
| `leftJoin(table, first, operator?, second)`                    | 別のテーブルとの左外部結合（joinクロージャをサポート） |
| `rightJoin(table, first, operator?, second)`                   | 別のテーブルとの右外部結合（joinクロージャをサポート） |
| `crossJoin(table)`                                             | 別のテーブルとのクロス結合 |
| `joinRaw(expression, bindings?)`                               | パラメータバインディング付きの生のjoin句 |
| `groupBy(columns)`                                             | グループ化カラム |
| `having(column, operator?, value?)`                            | グループ化結果に対するhaving条件 |
| `orHaving(column, operator?, value?)`                          | グループ化結果に対するhaving条件のor形式 |
| `havingRaw(sql, bindings?)`                                    | 生のSQL having条件（任意のバインディング付き） |
| `orHavingRaw(sql, bindings?)`                                  | 生のSQL or having条件（任意のバインディング付き） |
| `orderBy(column, direction='ASC')`                             | ソート（方向は`'ASC'`または`'DESC'`） |
| `orderByRaw(sql)`                                              | 生のSQL order by式 |
| `limit(n) / offset(n)`                                       | 取得行数上限およびスキップ数 |
| `clone()`                                                      | クエリビルダーインスタンスの独立した複製を生成 |
| `setHashFields(fields)`                                        | このビルダーで自動ハッシュ化するカラムをオーバーライド |

### ターミナルメソッド

これらはPromiseを返し基盤となるクエリを実行します（または実行せずにコンパイルされたクエリ文字列を返します）。

| メソッド | 引数 | 戻り値 |
| -------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| `toSql()`                                    | なし | `string`（実行せずにコンパイルされたselect SQL文字列） |
| `get()`                                      | なし | 一致する行の`Promise<Array<object>>`（一致がない場合は`[]`） |
| `first(where?)`                              | オプションの`{ column: value }`オブジェクト | `Promise<object\|null>`（最初の一致行、またはnull） |
| `firstWhere(col, op?, val?)`                 | カラム名、演算子または値、オプションの値 | `Promise<object\|null>`（最初の一致行、またはnull） |
| `latest(count?, column?)`                    | `null`または数値、オプションのカラム名 | countが`null`: `Promise<object\|null>`、数値count: `Promise<Array<object>>` |
| `exists()`                                   | なし | 一致する行が存在するかどうかの`Promise<boolean>` |
| `doesntExist()`                              | なし | 一致する行が存在しないかどうかの`Promise<boolean>` |
| `count(column?)`                             | オプションのカラム名（デフォルトは`*`） | `Promise<number>`（一致する行数。group byの場合はグループ数を返す） |
| `sum(column)`                                | カラム名 | `Promise<number>`（値の合計。行がない場合は0） |
| `avg(column)`                                | カラム名 | `Promise<number\|null>`（平均値。行がない場合はnull） |
| `min(column)`                                | カラム名 | `Promise<any>`（最小値。行がない場合はnull） |
| `max(column)`                                | カラム名 | `Promise<any>`（最大値。行がない場合はnull） |
| `pluck(column)`                              | カラム名 | 一致する行全体におけるそのカラム値の`Promise<Array>`（一致がない場合は空の`[]`） |
| `getWithCount()`                             | なし | `Promise<{ rows: Array<object>, total: number }>`（rowsからは内部の`_total_count`フィールドが除外されます） |
| `paginate(perPage?, pageName?)`              | `perPage`数値（デフォルト10）、オプションの`pageName`クエリパラメータ文字列（デフォルト`'page'`） | アイテム、合計件数、ナビゲーションリンクを含む`Promise<Paginator>`インスタンス |
| `insert(data)`                               | 行オブジェクト、または行オブジェクトの配列 | 単一行: `{ insertId, affectedRows, changedRows }`、行の配列: `affectedRows`と`insertId`を含む一括挿入ヘッダー |
| `update(data, where?)`                       | カラムと値のオブジェクト、オプションのwhereオブジェクト | `Promise<object>` `{ affectedRows, changedRows, insertId }` |
| `increment(column, amount?, extra?, where?)` | カラム名、オプションの数値（デフォルト1）、オプションの追加カラムオブジェクト、オプションのwhereオブジェクト | `Promise<object>` `{ affectedRows, changedRows, insertId }` |
| `decrement(column, amount?, extra?, where?)` | カラム名、オプションの数値（デフォルト1）、オプションの追加カラムオブジェクト、オプションのwhereオブジェクト | `Promise<object>` `{ affectedRows, changedRows, insertId }` |
| `save(data)`                                 | `update(data)`のエイリアス | `Promise<object>` `{ affectedRows, changedRows, insertId }` |
| `delete(where?)`                             | オプションのwhereオブジェクト（またはチェーンされた`where`） | `Promise<object>` `{ affectedRows, changedRows, insertId }`（whereが設定されていない場合はスロー） |

ビルダー自体はthenableであり非同期反復可能であるため直接`await DB.table('users')`や`for await (const row of DB.table('users'))`を使用できます。どちらの形式もselectを実行し同じ`Array<object>`形状を使用します。

<a name="結果の取得"></a>

## 結果の取得

<a name="すべての行の取得"></a>

### すべての行の取得

`get`メソッドはクエリを実行し結果オブジェクトの配列を返します。

```javascript
const users = await DB.table('users').get();
// users: Array<object>. empty [] when no rows match.
```

クエリビルダーは非同期反復可能でもあり`get`を呼び出さずにビルダーインスタンスを直接反復処理できます。

```javascript
for await (const user of DB.table('users')) {
  console.log(user.name);
}
// yields plain row objects (not model instances).
```

<a name="単一行の取得"></a>

### 単一行の取得

単一行のみを取得する必要がある場合は`first`メソッドを使用します。配列でラップせずにオブジェクトを直接返します。

```javascript
const user = await DB.table('users').where('email', 'tarou@example.com').first();
// user: object | null. null when no row matches.
```

<a name="最新レコードの取得"></a>

### 最新レコードの取得

`latest`メソッドは指定されたカラムの降順で行をソートし単一行または配列を返します。これは`orderBy(column, 'DESC').limit(n).get()`の短縮形です。

```javascript
const newest = await DB.table('posts').latest();
// newest: object | null
const recent = await DB.table('posts').latest(10);
// recent: Array<object>
const recentByPublished = await DB.table('posts').latest(10, 'published_at');
```

引数なしで`latest()`を呼び出すと`created_at`降順でソートされ最新の単一行（一致がない場合はnull）を返します。数値の`count`を渡すとその件数の行を配列として返します。カラムのデフォルトは`created_at`ですが2番目の引数でオーバーライドできます。

<a name="単一カラム値の取得pluck"></a>

### 単一カラム値の取得（pluck）

単一カラムの値を含むフラットな配列を取得したい場合は`pluck`メソッドを使用します。

```javascript
const titles = await DB.table('posts').pluck('title');
// titles: Array. one entry per matching row, in the order returned by the database.
```

<a name="集計countsumavgminmax"></a>

### 集計（count、sum、avg、min、max）

クエリビルダーはデータを集計するためのヘルパーメソッドを提供します: `count`、`sum`、`avg`、`min`、`max`。

```javascript
const totalOrders = await DB.table('orders').where('status', 'pending').count();
// totalOrders: number. 0 when no rows match.

const totalRevenue = await DB.table('orders').where('status', 'completed').sum('amount');
// totalRevenue: number. 0 when no rows match.

const avgPrice = await DB.table('products').where('category_id', 4).avg('price');
// avgPrice: number | null. null when no rows match.

const cheapest = await DB.table('products').min('price');
const priciest = await DB.table('products').max('price');
```

すべての集計メソッドは結合されたテーブル、テーブルエイリアス、where制約を自動的に尊重します:

```javascript
const totalDuration = await DB.table('track_interactions')
  .join('tracks', 'track_interactions.track_id', '=', 'tracks.id')
  .where('track_interactions.user_id', 1)
  .where('track_interactions.type', 'like')
  .sum('tracks.duration');
```

<a name="select句"></a>

## select句

デフォルトではクエリビルダーはすべてのカラムを選択します。特定のカラムを指定するには`select`メソッドを使用します。

```javascript
const users = await DB.table('users').select('id', 'name', 'email').get();
// users: Array<object>. each row only carries the requested columns.
```

select句に生のSQL式を挿入する必要がある場合は任意のバインディングとともに`selectRaw`メソッドを使用します。

```javascript
const users = await DB.table('users')
  .selectRaw('COUNT(id) as total, status')
  .groupBy('status')
  .get();
// users: Array<object>. each row has { total, status }.
```

クエリに重複のない結果のみを強制的に返させるには`distinct`メソッドを使用します。カラム名を指定して呼び出すとselectリストも同時に設定されます。

```javascript
const activeRoles = await DB.table('users').distinct('role').get();
// activeRoles: Array<object>. each row has the distinct role values.
```

<a name="サブクエリselect"></a>

### サブクエリselect

`selectSub`メソッドを使用してselect句にサブクエリを追加できます。クロージャまたはクエリビルダーインスタンスとエイリアス名を受け取ります。

```javascript
const users = await DB.table('users')
  .select('id', 'name')
  .selectSub(q => {
    q.from('orders').selectRaw('COUNT(*)').whereColumn('orders.user_id', 'users.id');
  }, 'orders_count')
  .get();
```

<a name="where句"></a>

## where句

<a name="基本のwhere句"></a>

### 基本のwhere句

`where`メソッドはカラム名、演算子、値の3つの引数を受け取ります。演算子を省略した場合ビルダーは等価（`=`）を前提とします。

```javascript
await DB.table('users').where('votes', '=', 100).get();
await DB.table('users').where('votes', 100).get();
await DB.table('users').where('votes', '>=', 100).get();
await DB.table('users').where('name', 'LIKE', '%test%').get();
```

`whereNot`メソッドは条件を否定します。

```javascript
await DB.table('users').whereNot('status', 'inactive').get();
```

オブジェクトを渡して複数の等価条件を同時に適用することもできます。

```javascript
await DB.table('users').where({
  status: 'active',
  role: 'admin'
}).get();
```

<a name="論理グループ化whereクロージャ"></a>

### 論理グループ化（whereクロージャ）

複雑なグループ化された論理式（ネストされた括弧）を作成するには`where`、`whereNot`、`orWhere`、`orWhereNot`にクロージャを渡します。クロージャは制約をネストするためのクエリビルダーインスタンスを受け取ります。

```javascript
const users = await DB.table('users')
  .where('active', 1)
  .where(q => {
    q.where('role', 'admin')
     .orWhere('role', 'superadmin');
  })
  .get();
// compiles to: WHERE `active` = ? AND (`role` = ? OR `role` = ?)
```

ネストされたクロージャを`whereNot`または`orWhereNot`と組み合わせて式を`NOT (式)`でラップできます:

```javascript
const users = await DB.table('users')
  .where('tenant_id', 1)
  .whereNot(q => {
    q.where('status', 'banned')
     .orWhere('suspended', 1);
  })
  .get();
```

<a name="or文"></a>

### or文

論理OR演算子で句をチェーンするには`orWhere`メソッドを使用します。

```javascript
await DB.table('users')
  .where('votes', '>', 100)
  .orWhere('name', 'tarou')
  .get();
```

否定のOR条件には`orWhereNot`メソッドも利用できます。

<a name="その他のwhere句"></a>

### その他のwhere句

クエリビルダーは一般的な条件タイプのための特殊なメソッドを提供します。

**whereIn / whereNotIn**
指定されたカラムの値が配列内に含まれているか、またはサブクエリクロージャに一致することを検証します。

```javascript
await DB.table('users').whereIn('id', [1, 2, 3]).get();
// when the array is empty, whereIn compiles to `0 = 1` (no rows)
// and whereNotIn compiles to `1 = 1` (all rows).
```

<a name="whereinでのサブクエリ"></a>

### whereInでのサブクエリ

`whereIn`および`whereNotIn`にクロージャコールバックを渡してサブクエリフィルターを構築できます:

```javascript
const customersWithOrders = await DB.table('users')
  .whereIn('id', q => {
    q.from('orders').select('user_id').where('status', 'paid');
  })
  .get();
// compiles to: WHERE `id` IN (SELECT `user_id` FROM `orders` WHERE `status` = ?)
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**
カラムの値がnullであること、またはnullでないことを検証します。

```javascript
await DB.table('users').whereNull('deleted_at').get();
await DB.table('users').where('status', 'active').orWhereNull('archived_at').get();
await DB.table('users').whereNotNull('verified_at').orWhereNotNull('confirmed_at').get();
```

**whereBetween / whereNotBetween / orWhereBetween / orWhereNotBetween**
カラムの値が2つの境界内にあることを検証します。厳密に2つの値を含む配列を指定する必要があります。両方の値が定義されている必要があります（境界をSQLのnullのままにするには`undefined`ではなく`null`を使用してください）。

```javascript
await DB.table('users').whereBetween('votes', [1, 100]).get();
await DB.table('orders')
  .whereBetween('total', [50, 200])
  .orWhereBetween('total', [500, 1000])
  .get();
```

**whereRaw / orWhereRaw**
パラメータ化されたバインディングを使用して生のSQL where句をクエリに挿入します:

```javascript
await DB.table('orders')
  .whereRaw('price > IF(state = "tx", ?, 100)', [200])
  .orWhereRaw('status = ?', ['pending'])
  .get();
```

<a name="whereexistsとwherenotexists"></a>

### whereExistsとwhereNotExists

`EXISTS (SELECT 句)`サブクエリを作成するには`whereExists`および`whereNotExists`（または対応する`orWhereExists` / `orWhereNotExists`）を使用します:

```javascript
const users = await DB.table('users')
  .whereExists(q => {
    q.from('orders')
      .whereColumn('orders.user_id', 'users.id')
      .where('orders.total', '>', 100);
  })
  .get();
```

<a name="カラム間比較"></a>

### カラム間比較

同じ行内の2つの異なるカラムの値を比較するには`whereColumn`および`orWhereColumn`メソッドを使用します。

```javascript
await DB.table('users').whereColumn('updated_at', '>', 'created_at').get();

await DB.table('users')
  .where('status', 'active')
  .orWhereColumn('last_login_at', '>', 'created_at')
  .get();
```

<a name="条件分岐句whenとunless"></a>

## 条件分岐句（whenとunless）

特定の条件が真の場合にのみクエリ句を適用したい場合があります。メソッドチェーンを崩すことなくクエリを条件付きで変更するには`when`および`unless`を使用します。

`when`メソッドは第1引数がtruthyと評価された場合に指定されたクロージャを実行します。オプションの第3引数のクロージャは条件がfalsyの場合に実行されます。

```javascript
const role = req.query?.role;
const sortBy = req.query?.sort;

const users = await DB.table('users')
  .when(role, (q, val) => q.where('role', val))
  .when(sortBy, (q, val) => q.orderBy(val, 'ASC'), q => q.orderBy('id', 'DESC'))
  .get();
```

`unless`メソッドは逆の動作をします。第1引数がfalsyの場合にクロージャを実行します。

```javascript
const users = await DB.table('users')
  .unless(includeInactive, q => q.where('active', 1))
  .get();
```

<a name="join結合"></a>

## join（結合）

クエリビルダーは内部結合、左/右外部結合、クロス結合、生の結合式を使用した複数テーブルの結合をサポートしています。

<a name="内部結合inner-join"></a>

### 内部結合（inner join）

基本的な内部結合を実行するには`join`メソッドを呼び出します。対象テーブル、1番目のカラム、任意の演算子（デフォルトは`=`）、2番目のカラムを指定できます。

```javascript
const users = await DB.table('users')
  .join('contacts', 'users.id', '=', 'contacts.user_id')
  .select('users.*', 'contacts.phone')
  .get();
```

引数が3つだけ渡された場合演算子はデフォルトで`=`になります:

```javascript
await DB.table('users')
  .join('contacts', 'users.id', 'contacts.user_id')
  .get();
```

<a name="左外部結合--右外部結合"></a>

### 左外部結合 / 右外部結合

`LEFT JOIN`または`RIGHT JOIN`を実行するには`leftJoin`または`rightJoin`を使用します:

```javascript
const tracks = await DB.table('tracks')
  .leftJoin('genres', 'tracks.genre_id', '=', 'genres.id')
  .select('tracks.*', 'genres.name as genre_name')
  .get();
```

複数の結合をチェーンしてリレーションを走査できます:

```javascript
const tracks = await DB.table('tracks')
  .leftJoin('track_artists', 'tracks.id', '=', 'track_artists.track_id')
  .leftJoin('artists', 'track_artists.artist_id', '=', 'artists.id')
  .select('tracks.*')
  .orderBy('artists.name', 'ASC')
  .get();
```

テーブル引数ではテーブルエイリアスがサポートされています:

```javascript
await DB.table('tracks')
  .leftJoin('artists as a', 'tracks.artist_id', '=', 'a.id')
  .get();
```

<a name="高度な結合句"></a>

### 高度な結合句

複数の結合条件を指定する必要がある場合や結合テーブルの`ON`句と`WHERE`句を組み合わせる必要がある場合は`join`、`leftJoin`、`rightJoin`の第2引数としてクロージャコールバックを渡します:

```javascript
const users = await DB.table('users')
  .join('contacts', j => {
    j.on('users.id', '=', 'contacts.user_id')
     .where('contacts.primary', '=', 1)
     .whereNull('contacts.deleted_at');
  })
  .get();
// compiles to: INNER JOIN `contacts` ON `users`.`id` = `contacts`.`user_id` AND `contacts`.`primary` = ? AND `contacts`.`deleted_at` IS NULL
```

結合句は`on`、`orOn`、`where`、`orWhere`、`whereNull`、`whereNotNull`をサポートしています。

<a name="クロス結合cross-join"></a>

### クロス結合（cross join）

デカルト積を実行するには`crossJoin`メソッドを使用します:

```javascript
const combos = await DB.table('sizes')
  .crossJoin('colors')
  .get();
```

<a name="生クエリによる結合raw-joins"></a>

### 生クエリによる結合（raw joins）

複雑な結合条件や結合内でのサブクエリには`joinRaw`を使用します:

```javascript
await DB.table('users')
  .joinRaw('LEFT JOIN contacts ON contacts.user_id = users.id AND contacts.status = ?', ['active'])
  .get();
```

<a name="ソートグループ化制限"></a>

## ソート、グループ化、制限

`orderBy`メソッドは結果セットをソートします。2番目の引数は方向を指定し`ASC`または`DESC`を受け入れます。

```javascript
await DB.table('users')
  .orderBy('name', 'DESC')
  .get();
```

`groupBy`メソッドは結果をグループ化するための1つ以上のカラム名を受け取ります。

```javascript
await DB.table('users')
  .groupBy('account_id', 'status')
  .get();
```

`having`および`havingRaw`メソッドはグループ化された結果をフィルタリングします。論理OR条件には`orHaving`および`orHavingRaw`を使用します:

```javascript
await DB.table('orders')
  .groupBy('account_id')
  .having('total', '>', 500)
  .orHaving('item_count', '>=', 10)
  .get();

await DB.table('orders')
  .groupBy('account_id')
  .havingRaw('SUM(price) > ?', [2500])
  .orHavingRaw('COUNT(id) > ?', [50])
  .get();
```

`orderByRaw`メソッドは任意のソート式を可能にします:

```javascript
await DB.table('users')
  .orderByRaw('FIELD(status, "active", "pending", "banned")')
  .get();
```

`limit`および`offset`メソッドは返されるレコード数を制限し開始位置を指定します。

```javascript
await DB.table('users')
  .offset(10)
  .limit(5)
  .get();
```

<a name="挿入"></a>

## 挿入

`insert`メソッドはデータベースに挿入するカラムと値のペアのオブジェクトを受け取ります。

```javascript
const result = await DB.table('users').insert({
  email: 'tarou@example.com',
  name: 'tarou'
});
// result: { insertId, affectedRows, changedRows }
// use result.insertId to read the assigned auto increment id.
```

<a name="一括挿入"></a>

### 一括挿入

`insert`メソッドにオブジェクトの配列を渡すとクエリビルダーは高度に最適化された単一の一括挿入文を実行し正規化された更新ヘッダーを返します。

```javascript
const result = await DB.table('users').insert([
  { email: 'tarou@example.com', name: 'tarou' },
  { email: 'satou@example.com', name: 'satou' }
]);
// result: { affectedRows, changedRows, insertId }
// result.affectedRows indicates total inserted rows.
// result.insertId indicates the first inserted auto increment id.
```

<a name="更新"></a>

## 更新

`update`メソッドは既存のレコードを更新します。変更するカラムとその新しい値を含むオブジェクトを受け取ります。以前にチェーンされたwhere句に一致するすべてのレコードに影響します。

```javascript
const result = await DB.table('users')
  .where('id', 1)
  .update({ votes: 1 });
// result: { affectedRows, changedRows, insertId: 0 }
// result.affectedRows tells you how many rows actually changed.
```

`update()`は更新を特定のバッチ行数に制限するためにチェーンされた`.limit(n)`句も受け入れます:

```javascript
await DB.table('users')
  .where('status', 'pending')
  .limit(100)
  .update({ status: 'processing' });
```

`save`メソッドは`update`のエイリアスとして機能し同じ更新ヘッダーを返します。

<a name="インクリメントとデクリメント"></a>

### インクリメントとデクリメント

クエリビルダーは数値カラムの値をインクリメントまたはデクリメントするための便利なメソッドも提供します。どちらのメソッドも少なくとも1つの引数（変更するカラム）を受け取ります。カラムをインクリメントまたはデクリメントする量を制御するために2番目の引数を渡すことができます（デフォルトは`1`）。オプションの3番目の引数を使用すると同じ文で更新する追加のカラムのオブジェクトを渡すことができます。

```javascript
// increment votes by 1
await DB.table('users').where('id', 1).increment('votes');

// increment credits by 5
await DB.table('users').where('id', 1).increment('credits', 5);

// increment credits by 10 and update a timestamp in the same statement
await DB.table('users').where('id', 1).increment('credits', 10, { updated_at: '2026-09-14' });

// decrement credits by 2
await DB.table('users').where('id', 1).decrement('credits', 2);
```

<a name="削除"></a>

## 削除

`delete`メソッドはテーブルからレコードを削除します。`delete()`は削除を特定のバッチサイズに制限するためにチェーンされた`.limit(n)`句をサポートしています。

> [!IMPORTANT]
> `delete()`を実行する前に少なくとも1つのwhere条件が必要です。偶発的なテーブル全削除を防ぐため制約なしで`delete()`を呼び出すと例外がスローされます。

```javascript
const result = await DB.table('users')
  .where('status', 'inactive')
  .limit(500)
  .delete();
// result: { affectedRows, changedRows, insertId: 0 }
// result.affectedRows tells you how many rows were removed.
```

> [!NOTE]
> `update()`および`delete()`は`.join()`句もサポートしています。mysqlは`UPDATE`または`DELETE`文において`LIMIT`と複数テーブルの`JOIN`操作の組み合わせを禁止している点に注意してください（実行すると診断エラーがスローされます）。postgresqlおよびsqliteでは文法コンパイラが制限または結合を組み合わせた更新をテーブルの主キーに対するサブクエリ`IN`式（`WHERE id IN (SELECT id FROM テーブル)`）に自動的にコンパイルしすべてのデータベースエンジン間で一貫した動作を保証します。

<a name="クエリの複製"></a>

## クエリの複製

`clone`メソッドはクエリビルダーインスタンスの独立したディープコピーを作成します。複製されたクエリはすべてのselectカラム、where句、結合定義、group byフィールド、having句、ソート条件、limit/offsetパラメータ、バインディング、ハッシュ設定を継承します。複製されたインスタンスを変更しても元のクエリには影響しません。

```javascript
const activeUsers = DB.table('users').where('status', 'active');

// branch query to compute total count without altering base query
const totalActive = await activeUsers.clone().count();

// branch query to fetch a paginated slice
const pageResults = await activeUsers.clone().limit(20).offset(0).get();
```

<a name="自動ハッシュ化"></a>

## 自動ハッシュ化

クエリビルダーは機密カラムを認識し挿入および更新時にbcryptを使用してその値を自動的にハッシュ化します。デフォルトでは`password`という名前のカラムのみがこの動作をトリガーします。

`config/app.js`で`app.hashFields`（またはカンマ区切りリストとしての`HASH_FIELDS`環境変数）を使用してアプリケーション全体で自動ハッシュ化されるフィールドを宣言するか`setHashFields`を使用して特定のクエリでオーバーライドできます。

```javascript
await DB.table('tokens')
  .setHashFields(['api_key'])
  .insert({ api_key: 'plain-text-key' });
// returns the single row insert result ({ insertId, affectedRows, changedRows }).
```

自動カラム設定に依存せずに任意の値をハッシュ化する必要がある場合は`dframework`からエクスポートされる`Hash`ファサードを使用します。

```javascript
import { Hash } from 'dframework';

// uses slow, secure bcrypt (for passwords)
await DB.table('users').insert({
  custom_secret: await Hash.make('plain-text-password')
});

// uses fast, peppered sha256 (for indexable tokens)
await DB.table('tokens').insert({
  token_hash: Hash.fast('plain-text-token')
});
```

カラムがハッシュフィールドとして設定されている場合は`whereHashed`および`orWhereHashed`メソッドを使用して平文の値に対して直接where比較を実行することもできます。

```javascript
const token = await DB.table('tokens').whereHashed('secret', 'plain-text-key').first();
// token: object | null. whereHashed hashes the value with sha256
// and compares it against the stored hash.
```
