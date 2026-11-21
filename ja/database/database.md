# データベース

- [データベース](#データベース)
  - [はじめに](#はじめに)
  - [設定](#設定)
  - [DBファサード](#dbファサード)
    - [メソッド一覧](#メソッド一覧)
  - [生クエリの実行](#生クエリの実行)
    - [パラメータ化クエリ](#パラメータ化クエリ)
    - [クエリのキャッシュ](#クエリのキャッシュ)
    - [戻り値の契約](#戻り値の契約)
    - [透過的クエリバッチ処理](#透過的クエリバッチ処理)
  - [トランザクション](#トランザクション)
  - [SQLヘルパー](#sqlヘルパー)
    - [raw式](#raw式)
    - [エスケープとサニタイズ](#エスケープとサニタイズ)

<a name="はじめに"></a>

## はじめに

dframeworkはmysql、postgresql、sqliteをサポートする統合データベース抽象化レイヤーを提供します。コアデータベースサービスは接続プーリング、クエリキャッシュ、透過的なselectバッチ処理、ダイアレクトコンパイル、セーブポイントトランザクション、パラメータバインディングを自動的に処理します。このドキュメントではデータベースの設定、生のSQLクエリの実行、組み込みSQLヘルパーの使用方法について説明します。流暢なクエリビルダーとアクティブレコードの実装については専用のドキュメントセクションで説明しています。

<a name="設定"></a>

## 設定

データベース設定は環境変数を通じて管理され`config/database.js`内の`database`名前空間のもとで`Config`ファサードを介して解決されます。フレームワークは`driver`（`mysql`、`pgsql`、`sqlite`）、`host`、`port`、`user`、`pass`、`name`、`connectionLimit`の値をサポートしています。

| オプション | 環境変数 | デフォルト値 | 説明 |
| ----------------- | --------------------- | ----------------------------------- | ------------------------------------------------ |
| `driver`          | `DB_DRIVER`           | `mysql`                             | データベースドライバー（`mysql`、`pgsql`、`sqlite`） |
| `host`            | `DB_HOST`             | `localhost`                         | データベースサーバーのホスト名 |
| `port`            | `DB_PORT`             | `3306` (mysql) / `5432` (pgsql)     | データベースサーバーのポート番号 |
| `user`            | `DB_USER`             | `root` (mysql) / `postgres` (pgsql) | データベースユーザー |
| `pass`            | `DB_PASS`             | `""`                                | データベースパスワード |
| `name`            | `DB_NAME`             | `dframework`                        | データベース名（またはsqliteファイルパス / `:memory:`） |
| `connectionLimit` | `DB_CONNECTION_LIMIT` | `10` (本番環境 `20`)                | 最大接続プールサイズ |
| `maxIdle`         | `DB_MAX_IDLE`         | `5`                                 | 最大アイドル接続数 |
| `idleTimeout`     | `DB_IDLE_TIMEOUT`     | `60000`                             | アイドル接続タイムアウト（ミリ秒） |

mysqlの場合接続プーリングにはキープアライブと10進数値（decimal）を有効にした`mysql2/promise`を使用します（`npm install mysql2`が必要）。postgresqlの場合接続プーリングには`pg`を使用し（`npm install pg`が必要）位置指定パラメータバインディング（`$1`、`$2`）とreturning句を自動的にコンパイルします。sqliteの場合フレームワークは組み込みの`node:sqlite`を使用し（node 22.5.0以上で利用可能）外部依存関係なしでwrite ahead logging（`WAL`）とbusyタイムアウトを自動的に有効にします。

接続プールまたはデータベース接続は最初のクエリ時に遅延生成されます（起動時に事前の接続は確立されません）。そのためデータベース設定に誤りがあってもプロセスの起動が妨げられることはありません。

<a name="dbファサード"></a>

## DBファサード

`DB`ファサードはアクティブなアプリケーションのプールされたデータベースインスタンスへのプロキシです。リクエストスコープの`app.db`、またはリクエストコンテキスト外で呼び出された場合はグローバルな`app.db`を解決します。アプリケーション起動前に`DB`を呼び出すと黙ってundefinedを返すのではなく明示的な診断エラーをスローします。

<a name="メソッド一覧"></a>

### メソッド一覧

`DB`ファサードは以下のメソッドを公開します。`戻り値`列には空の配列やnullのセマンティクスを含む実際の解決値を示します。

| メソッド | 引数 | 戻り値 |
| ---------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DB.query(sql, params?, options?)` | 生のパラメータ化SQL、params配列、`{ cache, ttl, _skipBatch }` | select文の場合は`Array<object>`（行がない場合は空の`[]`）、insert/update/deleteの場合は正規化された更新ヘッダー（`affectedRows`、`changedRows`、`insertId`を含む） |
| `DB.table(name)`                   | テーブル名文字列 | このテーブルにバインドされた`TableQuery`クエリビルダー（チェーン可能、決してnullにはならない） |
| `DB.insert(table, data)`           | テーブル名、カラムと値のオブジェクト | 正規化された更新ヘッダー（`result.insertId`または`result.affectedRows`を使用） |
| `DB.transaction(fn)`               | `async (conn) => result`                                          | コールバック`fn`が解決した結果（ネストされたセーブポイント`sp_1`、`sp_2`をサポートしロールバック後に再スロー） |
| `DB.inTransaction()`               | なし | `boolean`（現在の非同期実行コンテキストがアクティブなトランザクション内にある場合はtrue） |
| `DB.transactionDepth`              | なし | `number`（現在のトランザクションのネストレベル、外側の場合は0） |
| `DB.databaseExists()`              | なし | `Promise<boolean>` |
| `DB.ensureDatabaseExists()`        | なし | `Promise<void>`（`database.name`が欠けている場合は診断エラーをスロー） |
| `DB.ensureTableExists(table)`      | テーブル名 | `Promise<void>`（テーブルが存在しない場合はスロー、クエリビルダー内部で使用） |
| `DB.close()`                       | なし | `Promise<void>`（プールを終了し共有クエリキャッシュをクリア） |

`pool`ゲッターは基盤となるドライバプールまたはsqliteデータベースインスタンスに直接アクセスする必要がある場合にそれを返します。通常のコードではほとんど必要ありません。

<a name="生クエリの実行"></a>

## 生クエリの実行

生のSQL文を実行するには`query`メソッドを使用します。`DB`ファサードはインポート不要でアプリケーション全体でグローバルに利用可能です。

<a name="パラメータ化クエリ"></a>

### パラメータ化クエリ

SQLインジェクションを防ぐために常にパラメータ化クエリを使用してください。2番目の引数としてバインディングの配列を渡します。フレームワークは内部でプリペアドステートメントを使用します。

```javascript
const rows = await DB.query(
  'SELECT * FROM users WHERE status = ? AND age > ?',
  ['active', 18]
);
// rows: Array<object>, one entry per matching row. [] when no rows match.
```

<a name="クエリのキャッシュ"></a>

### クエリのキャッシュ

dframeworkにはメモリ制限付きのクエリキャッシュが含まれています。3番目の引数として設定オブジェクトを渡すことで特定のselectクエリで有効にできます。

```javascript
const rows = await DB.query(
  'SELECT * FROM settings WHERE scope = ?',
  ['global'],
  { cache: true, ttl: 1000 }
);
// rows: Array<object>. identical to the uncached return shape.
```

`ttl`パラメータはキャッシュの有効期間をミリ秒単位で定義します。デフォルトは500ミリ秒です。

キャッシュシステムは生のSQLを解析して参照されているテーブル名を抽出します。テーブルに対して`INSERT`、`UPDATE`、`DELETE`を実行するとフレームワークはそのテーブルを参照しているキャッシュ済みのselectクエリを自動的にフラッシュします。これにより手動で無効化することなく後続の読み取りの一貫性が保たれます。

キャッシュはLRU方式でありリクエストごとに`database.queryCache.maxEntries`件（デフォルト`200`）に制限されます。頻繁に使用されるクエリは読み取るだけで最新として保持され未使用のエントリを手動で破棄する必要はありません。

<a name="戻り値の契約"></a>

### 戻り値の契約

`query`メソッドの解決値はSQLステートメントのタイプに依存します。これは理解しておくべき最も重要な契約です。

| 文 | 戻り値 | 該当なし / 空 |
| ----------------------------------------------------- | -------------------------------------------- | ----------------------------------------------------------- |
| `SELECT` / `SHOW` / `DESCRIBE` / `EXPLAIN` / `PRAGMA` | `Array<object>`                              | `[]`                                                        |
| `WITH 句による SELECT (更新なし)`                     | `Array<object>`                              | `[]`                                                        |
| `INSERT`                                              | `{ affectedRows, changedRows, insertId }`    | `insertId`を保持（何も挿入されなかった場合は`affectedRows: 0`） |
| `UPDATE`                                              | `{ affectedRows, changedRows, insertId: 0 }` | 一致する行がなかった場合は`affectedRows: 0` |
| `DELETE`                                              | `{ affectedRows, changedRows, insertId: 0 }` | 一致する行がなかった場合は`affectedRows: 0` |

フレームワークはmysql、postgresql、sqlite間でクエリ実行結果を正規化し統一された契約にまとめます。常に（selectの場合は）行配列を（更新の場合は）正規化された更新ヘッダー`{ affectedRows, changedRows, insertId }`を受け取ります。更新ヘッダーはプレーンなオブジェクトであり`length`を持たないため配列チェックではなく`affectedRows`で判定してください。

<a name="透過的クエリバッチ処理"></a>

### 透過的クエリバッチ処理

selectは同じマイクロタスク内で発行された単純な等価検索やIN検索（`SELECT * FROM table WHERE column = ?`や`SELECT * FROM table WHERE column IN (?, ?, ?)`）を単一のIN句クエリに統合するリクエストごとのバッチャーを通過します。同一のパラメータ値は重複排除されます。

これは完全に透過的です。呼び出し元はあたかも自分自身のクエリを発行したかのように特定の値に一致する行を受け取ります。バッチャーは上記のパターンにのみ適用されこれより複雑なもの（JOIN、`AND status = ?`の混在、エイリアス指定された射影、`null`/`undefined`パラメータなど）はバッチ処理をバイパスして元のクエリを直接実行します。

意図的にバッチ処理をスキップする場合（バッチャー自身のコールバック内やカーソル依存のロジックなど）は`DB.query`のオプション引数に`{ _skipBatch: true }`を渡します。

> [!NOTE]
> クエリのバッチ処理はリクエストごとに管理されます。同一イベントループティック中に複数のコンポーネント、リレーション、ヘルパーが同じテーブルとカラムを照会すると`QueryBatcher`がそれらをプールしマイクロタスクキュー上でバッチ実行をスケジュールします。最大1024個の一意なキーが1回のデータベース往復にマージされ各呼び出し元のPromiseに行セットがマッピングされます。リクエストライフサイクルが終了すると並行リクエスト間での状態リークを防ぐためにバッチャーは閉じられます。

<a name="トランザクション"></a>

## トランザクション

アトミックな実行を保証するためにデータベース操作を`DB.transaction`でラップします。トランザクションは同じ非同期コンテキスト内のネストされたモデルクエリや生の`DB.query`呼び出し全体に専用の接続を自動的に伝播します:

```javascript
await DB.transaction(async (conn) => {
  await DB.table('users').where({ id: 1 }).update({ status: 'active' });
  await DB.table('audit_logs').insert({ user_id: 1, action: 'activated' });
});
```

ネストされた呼び出しは自動的にセーブポイント（`sp_1`、`sp_2`）を作成し内部ブロックが失敗した場合はセーブポイントを解放またはロールバックします。

アプリケーションサービスやイベントリスナー内でトランザクション状態を検査するには:

```javascript
if (DB.inTransaction()) {
  const depth = DB.transactionDepth;
}
```

<a name="sqlヘルパー"></a>

## SQLヘルパー

dframeworkには`SqlHelpers`エクスポート経由でアクセスできるSQL式のフォーマットと保護のための専用モジュールが含まれています。

<a name="raw式"></a>

### raw式

データベース関数を実行するために標準のパラメータエスケープをバイパスする必要がある場合があります。これは`raw`ヘルパーを使用して行います。

```javascript
import { SqlHelpers } from 'dframework';

await DB.table('users').update({ updated_at: SqlHelpers.raw('NOW()') }, { id: 1 });
```

> [!IMPORTANT]
> SQLインジェクションを防ぐため`SqlHelpers.raw()`は許可リストに基づいて動作し`CURRENT_TIMESTAMP`、`CURRENT_DATE`、`CURRENT_TIME`、`NOW()`、`UUID()`、`UUID_SHORT()`、`UNIX_TIMESTAMP()`、`UTC_TIMESTAMP()`、`UTC_DATE()`、`UTC_TIME()`、`RAND()`、`NULL`、`TRUE`、`FALSE`、`DEFAULT`のみを受け入れます。その他の式を渡すと即座に診断エラーがスローされます。

| ヘルパー | 引数 | 戻り値 |
| ---------------------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `SqlHelpers.raw(expr)`                               | 許可リストに含まれるSQL関数文字列 | `{ [Symbol.for('dframework.sqlRaw')]: true, value: string }`ラッパー（クエリビルダーが式をインライン化するために使用） |
| `SqlHelpers.escapeIdentifier(id)`                    | 識別子文字列 | バッククォートで囲まれた文字列（含まれるバッククォートは二重化） |
| `SqlHelpers.escapeTable(table, wrap?)`               | テーブル名またはエイリアス付きテーブル名 | エスケープされたテーブル式 |
| `SqlHelpers.escapeColumn(col, wrap?)`                | カラム名またはtable.column | エスケープされたカラム式 |
| `SqlHelpers.validateIdentifier(id, name?)`           | 識別子文字列、記述子文字列 | 検証済みの識別子文字列（不正な文字が含まれる場合はスロー） |
| `SqlHelpers.sanitizeTable(name)`                     | テーブル名 | サニタイズされたテーブル名文字列 |
| `SqlHelpers.sanitizeColumn(col)`                     | カラム名（オプションで`table.column`） | `[a-zA-Z0-9_]`以外の文字が除去された入力 |
| `SqlHelpers.sqlValue(val, booleanKeyword?, isAnsi?)` | 任意のJavaScript値 | 安全なSQLリテラル文字列 |

<a name="エスケープとサニタイズ"></a>

### エスケープとサニタイズ

`escapeIdentifier`メソッドはテーブル名やカラム名をバッククォートで囲み既存のバッククォートを安全にエスケープします。

```javascript
import { SqlHelpers } from 'dframework';

const column = SqlHelpers.escapeIdentifier('user`name'); // returns `user``name`
```

`sanitizeColumn`メソッドは英数字とアンダースコア以外のすべての文字をカラム名から除去します。結合テーブルの標準的なドット記法もサポートします。

```javascript
const safe = SqlHelpers.sanitizeColumn('users.first_name;'); // returns users.first_name
```

`sqlValue`メソッドは任意のJavaScript変数を生のSQLクエリに直接挿入しても安全な文字列にフォーマットします。文字列のエスケープ、真偽値の整数またはキーワードへの変換、nullの処理、`raw`ヘルパーによって生成された式のアンパックを行います。
