# マイグレーション

- [マイグレーション](#マイグレーション)
  - [はじめに](#はじめに)
  - [マイグレーションの生成](#マイグレーションの生成)
  - [テーブルの作成](#テーブルの作成)
  - [カラムタイプ](#カラムタイプ)
  - [カラム修飾子](#カラム修飾子)
  - [外部キー](#外部キー)
  - [テーブルの変更](#テーブルの変更)
  - [テーブルの削除](#テーブルの削除)
  - [マイグレーションの実行](#マイグレーションの実行)
    - [マイグレーションステータス](#マイグレーションステータス)
    - [ロールバック](#ロールバック)
    - [トランザクションマイグレーションとDDL文](#トランザクションマイグレーションとddl文)

<a name="はじめに"></a>

## はじめに

マイグレーションはデータベースのバージョン管理のようなものでありチームがアプリケーションのデータベーススキーマ定義を定義および共有できるようにします。同僚にローカルデータベーススキーマへ手動でカラムを追加するよう依頼した経験があるならマイグレーションが解決する問題に直面したことがあるはずです。

各マイグレーションファイルは非同期の`up`関数をエクスポートする必要がありオプションで`down`関数もエクスポートできるESモジュールです。フレームワークは`Schema`サービス、`table`クエリビルダー、生の`db`接続、`query`ヘルパーを含むコンテキストオブジェクトを自動的に注入します。これらのプロパティはマイグレーションの実行中グローバルでも利用可能です。

<a name="マイグレーションの生成"></a>

## マイグレーションの生成

新しいマイグレーションを作成するにはマイグレーション名を指定して`migrate:make` CLIコマンドを使用します:

```bash
dstrn migrate:make create_users_table
dstrn migrate:make add_role_to_users_table
```

これによりタイムスタンプ付きのマイグレーションファイルが`database/migrations/`にスキャフォールドされスタブ化された`up`関数と`down`関数が含まれます。ジェネレーターは一般的な命名パターン（`create_<table>_table`や`add_<col>_to_<table>_table`など）から対象テーブル名を自動的に推論します（`users`のようなテーブル名単体も指定できます）。

<a name="テーブルの作成"></a>

## テーブルの作成

新しいデータベーステーブルを作成するには`Schema`ファサードの`create`メソッドを使用します。`create`メソッドは2つの引数を受け取ります。テーブル名と新しいテーブルを定義するために使用される`TableBuilder`オブジェクトを受け取るクロージャです。

```javascript
export async function up({ Schema }) {
  await Schema.create('users', table => {
    table.increments('id');
    table.string('name');
    table.string('email').unique();
    table.timestamps();
  });
}

export async function down({ Schema }) {
  await Schema.dropIfExists('users');
}
```

<a name="カラムタイプ"></a>

## カラムタイプ

テーブルビルダーにはテーブル構築時に使用できるさまざまなカラムタイプが用意されています。

| タイプ | 説明 |
| ----------------------------------------------- | ----------------------------------------------- |
| `table.id('id');`                               | 自動インクリメントのbig integer主キー |
| `table.increments('id');`                       | 自動インクリメントのinteger主キー |
| `table.bigIncrements('id');`                    | 自動インクリメントのbig integer主キー |
| `table.integer('votes');`                       | integerカラム |
| `table.bigInteger('views');`                    | big integerカラム |
| `table.float('amount');`                        | floatカラム |
| `table.decimal('balance', '10,2');`             | 精度指定付きdecimalカラム |
| `table.boolean('confirmed');`                   | tinyint booleanカラム |
| `table.string('name', 100);`                    | 長さ指定可能なvarcharカラム |
| `table.text('description');`                    | textカラム |
| `table.json('options');`                        | jsonカラム |
| `table.enum('status', ['active', 'inactive']);` | enumカラム |
| `table.datetime('created_at');`                 | datetimeカラム |
| `table.date('birthday');`                       | dateカラム |
| `table.time('sunrise');`                        | timeカラム |
| `table.timestamps();`                           | created_atおよびupdated_atのdatetimeカラムを追加 |

<a name="カラム修飾子"></a>

## カラム修飾子

上記のカラムタイプに加えてデータベーステーブルにカラムを追加する際に使用できるいくつかのカラム修飾子があります。

| タイプ | 説明 |
| ------------------------------------------------------ | --------------------------- |
| `table.string('email').nullable();`                    | null値を許可 |
| `table.string('title').notNullable();`                 | null値を禁止 |
| `table.integer('votes').unsigned();`                   | integerをunsignedに設定 |
| `table.integer('status').defaultTo(1);`                | デフォルト値を設定 |
| `table.string('email').unique();`                      | ユニークインデックスを追加 |
| `table.unique('email', 'uniq_email');`                 | カラムにユニークインデックスを追加 |
| `table.unique(['org_id', 'slug'], 'uniq_org_slug');`   | 複合ユニークインデックスを追加 |
| `table.string('slug').index();`                        | 基本インデックスを追加 |
| `table.index(['user_id', 'status'], 'idx_user_stat');` | 複合名前付きインデックスを追加 |
| `table.integer('id').primary();`                       | 明示的に主キーを追加 |
| `table.string('status').comment('the user status');`   | カラムコメントを追加 |

<a name="外部キー"></a>

## 外部キー

dframeworkはテーブルへの外部キー制約の追加をサポートしています。`foreign`メソッドでローカルカラムを定義し`references`メソッドで参照先カラムとテーブルを指定しオプションでカスケードルールを定義します。

```javascript
export async function up({ Schema }) {
  await Schema.create('posts', table => {
    table.increments('id');
    table.integer('user_id').unsigned();
    
    table.foreign('user_id')
         .references('id', 'users')
         .onDelete('cascade')
         .onUpdate('cascade');
  });
}
```

<a name="テーブルの変更"></a>

## テーブルの変更

`Schema`ファサードの`table`メソッドを使用すると既存のテーブルを更新できます（カラムの追加、カラム定義の変更、カラムの名前変更、カラムの削除、インデックスや外部キーの管理）。

```javascript
export async function up({ Schema }) {
  await Schema.table('users', table => {
    // add a new column
    table.string('phone').nullable();

    // modify an existing column definition (fluent syntax)
    table.string('email', 150).nullable().modify();

    // modify an existing column definition (direct syntax)
    table.modify('bio', 'TEXT', { nullable: true });

    // rename a column
    table.renameColumn('username', 'handle');

    // drop a single column or multiple columns
    table.dropColumn('temp_token');
    table.dropColumns('old_field_a', 'old_field_b');

    // add indexes
    table.index('phone');
    table.index(['organization_id', 'role'], 'idx_users_org_role');
  });
}

export async function down({ Schema }) {
  await Schema.table('users', table => {
    table.dropForeign('fk_users_org_id');
    table.dropIndex('idx_users_org_role');
    table.renameColumn('handle', 'username');
  });
}
```

また`rename`メソッドを使用して既存のテーブル名を変更することもできます。

```javascript
await Schema.rename('old_table', 'new_table');
```

> [!NOTE]
> mysqlおよびpostgresqlは既存のカラムタイプの変更（`table.modify()`）や外部キーの変更をサポートしていますがsqliteは既存テーブルに対するカラム定義の変更、外部キーの追加または削除、主キーの変更をネイティブではサポートしていません。sqliteでこれらの操作を実行しようとするとテーブルを再作成して既存レコードを移行するための具体的なガイダンスを含む診断エラーが即座にスローされます。

<a name="テーブルの削除"></a>

## テーブルの削除

既存のテーブルを削除するには`drop`または`dropIfExists`メソッドを使用します。

```javascript
await Schema.drop('users');
await Schema.dropIfExists('users');
```

<a name="マイグレーションの実行"></a>

## マイグレーションの実行

保留中のマイグレーションを実行するには`migrate`コマンドを実行します:

```bash
dstrn migrate
```

フレームワークはmigrationsディレクトリ内のファイル名に基づきアルファベット順にマイグレーションを順番に実行します。どのマイグレーションがすでに実行されたかを追跡しバッチごとに整理するための`migrations`というデータベーステーブルが自動的に生成されます。

本番環境（`APP_ENV=production`）で実行する場合このコマンドは変更を適用する前に確認を求めます。自動デプロイパイプラインで非対話的にマイグレーションを実行するには`--force`フラグを渡します:

```bash
dstrn migrate --force
```

<a name="マイグレーションステータス"></a>

### マイグレーションステータス

どのマイグレーションが適用済みでどれが保留中かを確認するには以下を実行します:

```bash
dstrn migrate:status
```

これによりすべてのマイグレーションファイルと適用されたバッチ番号を一覧表示するステータステーブルが出力されます。

<a name="ロールバック"></a>

### ロールバック

最新のデータベース変更を取り消す必要がある場合は以下を実行します:

```bash
dstrn migrate:rollback
```

フレームワークは`migrations`テーブルを照会して最新のバッチ番号を特定しそれらのファイルの`down`関数を逆時系列順に実行します。

本番環境（`APP_ENV=production`）で実行する場合このコマンドはロールバックを適用する前に確認を求めます。自動パイプラインで非対話的に実行するには`--force`を渡します:

```bash
dstrn migrate:rollback --force
```

<a name="トランザクションマイグレーションとddl文"></a>

### トランザクションマイグレーションとDDL文

マイグレーションはエラー発生時に変更がロールバックされるようデータベーストランザクション内で実行されます。ただしmysqlはDDL文（`create table`、`alter table`、`drop table`など）を暗黙的にコミットします。マイグレーションに複数のDDL文が含まれており途中で失敗した場合失敗前に実行された文はmysqlでコミットされたままとなりマイグレーションのエントリは`migrations`テーブルに記録されません。マイグレーションは単一の責務に集中させDDLマイグレーションが失敗した場合はデータベーススキーマを確認してください。
