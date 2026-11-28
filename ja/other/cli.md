# cli

- [cli](#cli)
  - [はじめに](#はじめに)
  - [利用可能なコマンド](#利用可能なコマンド)
    - [スキャフォールディング](#スキャフォールディング)
    - [データベース](#データベース)
    - [ジェネレーター](#ジェネレーター)
    - [システム](#システム)
    - [ネイティブ](#ネイティブ)
    - [アーキテクチャ](#アーキテクチャ)
  - [プロジェクトの作成](#プロジェクトの作成)
  - [サーバーの実行](#サーバーの実行)
  - [tinker](#tinker)
  - [テストの実行](#テストの実行)
  - [テストの生成](#テストの生成)
  - [データベースアドバイザー](#データベースアドバイザー)
  - [スケジュールタスクと手動実行](#スケジュールタスクと手動実行)
  - [コマンドの作成](#コマンドの作成)
  - [コマンドの実行](#コマンドの実行)
  - [ジョブのディスパッチ](#ジョブのディスパッチ)
  - [キューワーカーの実行](#キューワーカーの実行)
  - [ログ管理](#ログ管理)
  - [デプロイと証明書](#デプロイと証明書)
    - [アプリケーションのデプロイ](#アプリケーションのデプロイ)
    - [証明書の管理](#証明書の管理)
  - [アーキテクチャアナライザー](#アーキテクチャアナライザー)
    - [依存関係グラフ](#依存関係グラフ)
    - [アーキテクチャlint](#アーキテクチャlint)
    - [デッドコード検出](#デッドコード検出)
    - [プロジェクトdoctor](#プロジェクトdoctor)
    - [翻訳lint](#翻訳lint)
  - [コンパイラ](#コンパイラ)
    - [javascriptコンパイル](#javascriptコンパイル)
    - [cssコンパイル](#cssコンパイル)

<a name="はじめに"></a>

## はじめに

dframeworkには`dstrn`バイナリ経由でアクセス可能なコマンドラインインターフェースが含まれています。新しいプロジェクトのスキャフォールディング、データベースの管理、アプリケーションファイルの生成、アーキテクチャの分析、および対話型シェルの実行を行うコマンドを提供します。

`dstrn init`、`dstrn help`（または`--help`）、`dstrn version`（または`-v`、`--version`）を除くすべてのコマンドは制限されており`config/app.js`を含む有効なdframeworkプロジェクトディレクトリ内での実行が必要です。

```bash
dstrn help
dstrn help serve
dstrn migrate --help
dstrn --version
```

`dstrn help <command>`にコマンド名を渡すか任意のコマンドに`--help` / `-h`を付加するとその特定のコマンドに関する使用法、引数、オプションが表示されます。

<a name="利用可能なコマンド"></a>

## 利用可能なコマンド

<a name="スキャフォールディング"></a>

### スキャフォールディング

| コマンド | 説明 |
| --------------------- | ------------------------------------------------------- |
| `dstrn init <dir>`    | 最小限のプロジェクトをスキャフォールディング |
| `dstrn serve`         | 現在のディレクトリでサーバーを実行 |
| `dstrn test [target]` | nodeテストランナーでアプリケーションテストを実行 |
| `dstrn tinker [code]` | 対話型replシェルを開く、またはコードを直接実行 |
| `dstrn key:generate`  | ランダムな暗号化キーを生成 |

<a name="データベース"></a>

### データベース

| コマンド | 説明 |
| ---------------------------- | ------------------------------------------------------- |
| `dstrn migrate [--schema-path=] [--force]` | 保留中のすべてのマイグレーションを実行 |
| `dstrn migrate:status`       | マイグレーションの状態を表示 |
| `dstrn migrate:rollback`     | 最後のマイグレーションバッチをロールバック |
| `dstrn migrate:squash [--schema-path=] [--prune] [--force]` | 適用済みマイグレーションを単一のスキーマスナップショットへ統合 |
| `dstrn migrate:make <name>`  | 新しいマイグレーションテンプレートを作成（命名パターンからテーブル名を推論） |
| `dstrn seed [name]`          | データベースシーダーを実行 |
| `dstrn seed:make <Name>`     | 新しいシーダーテンプレートを作成 |
| `dstrn drop [--force]`       | 本番環境以外で確認（または`--force`）付きでデータベース内のすべてのテーブルを削除（本番環境では常に無効） |
| `dstrn advisor [--apply] [--force]` | インデックスのないリレーションを検査しマイグレーションを生成 |

<a name="ジェネレーター"></a>

### ジェネレーター

| コマンド | 説明 |
| ------------------------------ | ---------------------------- |
| `dstrn make:model <Name>`      | 新しいモデルを作成 |
| `dstrn make:factory <Name> [--model=]` | 新しいモデルファクトリを作成 |
| `dstrn make:controller <Name>` | 新しいコントローラーを作成 |
| `dstrn make:middleware <Name>` | 新しいミドルウェアを作成 |
| `dstrn make:command <Name>`    | 新しいコンソールコマンドを作成 |
| `dstrn make:job <Name>`        | 新しいバックグラウンドジョブを作成 |
| `dstrn make:component <Name>`  | `public/js/components`に新しいクライアントwebコンポーネント（`dComponent`）を作成 |
| `dstrn make:test <Name>`       | 新しいテストファイルを作成（`--feature`、`--browser`、`--unit`） |

<a name="システム"></a>

### システム

| コマンド | 説明 |
| --------------------------------------- | ----------------------------------------------------------------------------- |
| `dstrn run <CommandName>`               | `console/commands`からユーザーコマンドを実行 |
| `dstrn queue:work [--queue=] [--driver=] [--once]` | バックグラウンドキュージョブをデーモンとして処理 |
| `dstrn queue:status [--queue=] [--driver=] [--json]` | ワーカープールの状態、保留ジョブ、失敗、バックオフを表示 |
| `dstrn schedule:list [--json]`          | 登録済みスケジュールタスク、間隔、次回実行、前回ステータスを表示 |
| `dstrn dispatch <JobName> [--payload=] [--tracking-id=]` | オプションのペイロードおよび関連付けidを使用して現在のプロセスでジョブを直接実行 |
| `dstrn logs:clear`                     | 日付フィルタリング付きでログファイルをクリアまたはアーカイブ |
| `dstrn deploy`                         | 本番デプロイおよびサービス設定を生成 |
| `dstrn cert <action>`                  | tls証明書を管理（`obtain`、`renew`、`status`） |
| `dstrn cert:obtain`                    | let's encryptのssl証明書を取得 |
| `dstrn cert:renew`                     | 期限切れの近いssl証明書を更新 |
| `dstrn cert:status`                    | 証明書の有効性と有効期限の詳細を表示 |
| `dstrn docs:publish`                   | プロジェクトにフレームワークドキュメントを公開 |

<a name="ネイティブ"></a>

### ネイティブ

| コマンド | 説明 |
| ----------------------------------- | --------------------------------------- |
| `dstrn simulate --<platform>`       | プラットフォームシミュレーターで実行 |
| `dstrn build --<platform>`          | 本番バイナリをビルド |
| `dstrn make:plugin <Name>`          | クロスプラットフォームのネイティブプラグインをスキャフォールディング |
| `dstrn native:status`               | ネイティブ設定と準備状態を表示 |
| `dstrn native:doctor [--<platform>]` | ネイティブ環境を診断（特定の対象を必須化する場合は`--desktop`、`--android`、`--ios`を指定） |
| `dstrn native:clean [--<platform>]` | ネイティブビルドアーティファクトをクリーンアップ |

<a name="アーキテクチャ"></a>

### アーキテクチャ

| コマンド | 説明 |
| ----------------- | ---------------------------------------- |
| `dstrn graph`     | アプリケーションの依存関係グラフを表示 |
| `dstrn lint`      | アーキテクチャ違反を検出 |
| `dstrn dead`      | 到達不能なデッドコードを検出 |
| `dstrn doctor`    | プロジェクトの品質スコアを算出 |
| `dstrn lang:lint` | 翻訳パリティとビューキーをチェック |

<a name="プロジェクトの作成"></a>

## プロジェクトの作成

`init`コマンドは標準のディレクトリ構造、デフォルト設定ファイル、サンプルのルート、および基本的なビューを含む新しいプロジェクトをスキャフォールディングします。

```bash
dstrn init my-app
```

<a name="サーバーの実行"></a>

## サーバーの実行

`serve`コマンドは現在のディレクトリでアプリケーションサーバーを起動します。

```bash
dstrn serve
```

これによりアプリケーション設定がロードされすべてのルートファイルがインポートされソケットサーバーが起動しスケジューラーが開始されhttpリクエストの待受が開始されます。起動バナーにはアプリケーションのurlとポートが表示されます。

`--host`および`--port`を使用してカスタムホストまたはポートを指定できます:

```bash
dstrn serve --host=0.0.0.0 --port=3000
```

値を指定せずに単独フラグとして`--host`を指定するとサーバーを`0.0.0.0`にバインドしてローカルネットワークに公開します:

```bash
dstrn serve --host
```

<a name="tinker"></a>

## tinker

`tinker`コマンドは対話型replシェルを提供するかすべてのモデルと完全なアプリケーションコンテキストを事前ロードした状態でインラインコードを直接実行します。

### 対話型シェル

引数なしでコマンドを実行して対話型セッションを開始します:

```bash
dstrn tinker
```

```
dframework> const users = await User.all();
dframework> users.length
42
dframework> await User.where('role', 'admin').get();
```

シェルはコマンド履歴をプロジェクトルートの`.dframework_history`に永続化します。すべてのフレームワークファサード（`DB`、`Config`、`Session`、`Auth`、`Log`、`Storage`、`Route`など）、モデル、およびモデルファクトリが事前ロードされます。replはタブ補完および組み込みのドットコマンド（`.models`、`.factories`、`.facades`、`.tables`、`.help`、`.exit`）を提供します:

```
dframework> .models
User, Post
dframework> .tables
users, posts, migrations
dframework> User.fac<tab>
dframework> User.factory().create()
```

### インラインコード実行

位置引数としてコードを直接渡すか`--execute`（`-e`）フラグを使用して対話型セッションを開始せずにコードを評価しフォーマットされた結果を即座に出力します:

```bash
dstrn tinker "await User.count()"
dstrn tinker "await DB.table('users').where('status', 'active').get()"
dstrn tinker --execute="await User.find(1)"
dstrn tinker -e "const u = await User.first(); return u.email;"
```

インライン実行はトップレベルのawait式と複数文ブロックをサポートします。`models/`内のすべてのモデルは実行開始前にグローバルスコープにロードされます。

<a name="テストの実行"></a>

## テストの実行

`test`コマンドは、ノード組み込みのテストランナーを使用して、自動環境分離（`APP_ENV=testing`）のもとでテストスイートを実行します。`tests/`配下の`*.test.js`に一致するすべてのテストファイルを自動検出します。テストの実行にはプロジェクトルートに`.env.testing`が存在することが必須です:

```bash
dstrn test
dstrn test tests/feature/UserTest.test.js
dstrn test --watch
dstrn test --only
dstrn test --raw
dstrn test --reporter=./custom-reporter.js
```

<a name="テストの生成"></a>

## テストの生成

`make:test`コマンドは、適切なインポートとライフサイクルのアサーションを備えたテストファイルをスキャフォールドします。

```bash
dstrn make:test UserTest
dstrn make:test LoginTest --browser
dstrn make:test CalculatorTest --unit
```

<a name="データベースアドバイザー"></a>

## データベースアドバイザー

`advisor`コマンドはデータベーススキーマ、カラム定義、アクティブなクエリ履歴を検査し外部キー、中間テーブル、頻繁にフィルタリングされるカラムの組み合わせにおける欠落インデックスを検出します。

```bash
dstrn advisor
```

```
 INFO  database index advisor

  tracks .....................................................  WARN  [idx_tracks_album_id]
    columns: album_id
    reason:  foreign key column album_id lacks leading index

  track_artists ..............................................  WARN  [idx_track_artists_track_id_artist_id]
    columns: track_id, artist_id
    reason:  frequently filtered column pair (track_id, artist_id) lacks composite index

 OK  found 2 recommendations (run "dstrn advisor --apply" to generate and execute migrations) (12.30ms)
```

推奨されるすべてのインデックスを含むタイムスタンプ付きマイグレーションを自動的にスキャフォールディングして実行するには`--apply`フラグを渡します:

```bash
dstrn advisor --apply
```

```
 INFO  database index advisor

  tracks .....................................................  WARN  [idx_tracks_album_id]
    columns: album_id
    reason:  foreign key column album_id lacks leading index

  track_artists ..............................................  WARN  [idx_track_artists_track_id_artist_id]
    columns: track_id, artist_id
    reason:  frequently filtered column pair (track_id, artist_id) lacks composite index

  2026_10_28_120000_add_idx_tracks_album_id.js .................  OK  [generated]
  2026_10_28_120001_add_idx_track_artists_track_id_artist_id.js ..  OK  [generated]

 INFO  running migrations

  2026_10_28_120000_add_idx_tracks_album_id.js .................  OK  [batch: 1] (4.50ms)
  2026_10_28_120001_add_idx_track_artists_track_id_artist_id.js ..  OK  [batch: 1] (3.80ms)

 OK  migrated 2 migrations (10.20ms)

 OK  all migrations applied successfully (18.50ms)
```

生成された各マイグレーションファイルは`table.index()`および`table.dropIndex()`呼び出しを含む`up`および`down`関数をエクスポートし`dstrn migrate:rollback`による完全なロールバックサポートを可能にします。

<a name="スケジュールタスクと手動実行"></a>

## スケジュールタスクと手動実行

`console/commands`に配置されるスケジューラーコマンドを作成できます。これらのタスクは設定されたスケジュールで自動的に実行されるほか`dstrn run`を使用してオンデマンドで手動実行することもできます。

<a name="コマンドの作成"></a>

### コマンドの作成

`make:command`ジェネレーターを使用して新しいコマンドファイルをスキャフォールディングします:

```bash
dstrn make:command PruneExpiredSessions
```

これにより`console/commands/PruneExpiredSessionsCommand.js`が作成されます（ジェネレーターは省略時に自動的に`Command`サフィックスを付加します）。生成されたファイルは実行ロジックを記述する`async handle(args, options)`メソッドを持つデフォルトクラスをエクスポートします。タスクはすべてのフレームワークグローバル（`DB`、`Log`、`Config`、モデル）に直接アクセスできます:

```javascript
export default class PruneExpiredSessionsCommand {
  async handle(args, options) {
    const isDryRun = this.hasOption('dry-run');
    const batchSize = parseInt(this.option('batch', 500), 10);
    const targetTable = this.argument(0, 'sessions');

    if (isDryRun) {
      Log.info(`previewing session cleanup for ${targetTable} in batches of ${batchSize}`);
      return;
    }

    const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    await DB.table(targetTable).where('updated_at', '<', cutoff).delete();
    Log.info('expired sessions pruned');
  }
}
```

<a name="コマンドの実行"></a>

### コマンドの実行

`run`コマンドを使用してターミナルからオンデマンドで任意のコマンドを実行できます:

```bash
dstrn run PruneExpiredSessions
```

シェルのタブ補完を容易にするため`.js`で終わるファイル名も受け付けられます:

```bash
dstrn run PruneExpiredSessions.js
```

手動実行時にパラメータを渡すにはタスク名の後に指定します。オプションとフラグは位置引数とともに`--key value`または`--key=value`形式で渡すことができます:

```bash
dstrn run PruneExpiredSessions sessions --batch=1000 --dry-run
# or
dstrn run PruneExpiredSessions sessions --batch 1000 --dry-run
```

dframeworkはcli引数を自動的にパースしコマンドインスタンスにヘルパーメソッドを注入します:

| メソッド / プロパティ | 説明 |
| -------------------------------- | -------------------------------------------------------------------- |
| `this.option('name', fallback)`  | `--name value`または`--name=value`からオプション値を取得 |
| `this.hasOption('name')`         | `--dry-run`などのフラグが渡された場合にboolean trueを返す |
| `this.argument(index, fallback)` | 0から始まるインデックスで位置引数を取得 |
| `this.options`                   | パースされたすべてのオプションとフラグを含む辞書 |
| `this.arguments`                 | すべての位置引数を含む配列 |
| `options`（第2パラメータ） | `handle(args, options)`に直接渡されるパース済みオプションオブジェクト |

dframeworkは`console/commands/PruneExpiredSessionsCommand.js`（または`PruneExpiredSessions.js`）からクラスをインポートしてインスタンス化し`handle(args, options)`を呼び出し完了時に経過実行時間を報告します。

`console/commands`内のタスクはスケジューラーによっても使用されます。`console/Schedule.js`でスケジュール設定された場合スケジューラーは`handle([], {})`を呼び出し`this.option()`は設定されたフォールバック値を安全に返します。

<a name="ジョブのディスパッチ"></a>

### ジョブのディスパッチ

`dstrn dispatch`を使用してコマンドラインからオンデマンドでジョブを直接実行できます（キューイングを行わず現在のプロセスで同期的に実行されます）:

```bash
dstrn dispatch CleanupExpiredSessionsJob
```

シェルのタブ補完を容易にするため`.js`で終わるファイル名も受け付けられます:

```bash
dstrn dispatch CleanupExpiredSessionsJob.js
```

`--payload`フラグを使用してjsonペイロードを渡すことができます:

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}'
```

また`--tracking-id`フラグを使用して明示的な関連付け識別子を割り当てることもできます:

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}' --tracking-id 'custom-correlation-id'
```

<a name="キューワーカーの実行"></a>

### キューワーカーの実行

キュー内のバックグラウンドジョブを処理する専用ワーカーデーモンを実行できます。

```bash
dstrn queue:work
```

キューの指定、ドライバーの上書き、または単一ジョブのみを実行して終了することも可能です。

```bash
dstrn queue:work --queue=emails
dstrn queue:work --driver=database
dstrn queue:work --once
```

<a name="ログ管理"></a>

## ログ管理

`logs:clear`コマンドは日付範囲フィルタリング、環境ターゲティング、アーカイブ、プレビューモードを備えた柔軟なログファイル管理を提供します:

```bash
dstrn logs:clear
dstrn logs:clear --env=prod
dstrn logs:clear --before=2026-01-01
dstrn logs:clear --after=2026-05-01 --archive
dstrn logs:clear --preview
```

| フラグ | 説明 |
| --------------------------- | ------------------------------------------------- |
| `--env=prod`または`--env=dev` | 特定の環境ログディレクトリを対象にする |
| `--before=YYYY-MM-DD`       | 指定日より前のログをクリア |
| `--after=YYYY-MM-DD`        | 指定日より後のログをクリア |
| `--archive`                 | 一致した各ログを`archives`サブディレクトリ内に`.gz`形式で個別圧縮し元のファイルを削除 |
| `--preview`                 | 実際の処理を行わずに影響を受けるファイルを表示 |

<a name="デプロイと証明書"></a>

## デプロイと証明書

<a name="アプリケーションのデプロイ"></a>

### アプリケーションのデプロイ

`deploy`コマンドはネイティブtlsを使用したベアメタルデプロイ（デフォルト）、caddyを使用したリバースプロキシ（`--proxy`）、またはcloudflareを使用したエッジルーティング（`--edge`）のための本番設定を生成します。

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com
dstrn deploy --proxy --domain=api.example.com
dstrn deploy --edge --domain=api.example.com
dstrn deploy --domain=api.example.com --email=admin@example.com --dry-run
sudo dstrn deploy --domain=api.example.com --email=admin@example.com --install
```

| フラグ | 説明 |
| -------------------- | ----------------------------------------------------------------------------- |
| `--domain=<domains>` | カンマ区切りの対象ドメイン |
| `--email=<email>`    | let's encrypt証明書のacme登録メールアドレス |
| `--proxy`            | 直接tlsの代わりにcaddyリバースプロキシ設定を生成 |
| `--edge`             | cloudflareガイダンス付きの平文http設定を生成 |
| `--port=<port>`      | カスタムアプリケーションポート（デフォルトは8250、直接tlsの場合は443） |
| `--user=<user>`      | systemdサービスのシステムユーザー（デフォルトは現在のユーザー） |
| `--dir=<dir>`        | 作業ディレクトリのパス |
| `--dry-run`          | ディスクに書き込まずに生成される設定をコンソールでプレビュー |
| `--install`          | 設定ファイルをシステムパスに直接書き込む（root権限が必要） |

<a name="証明書の管理"></a>

### 証明書の管理

`cert`コマンドファミリーは証明書の発行、更新、状態監視を処理します。

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com
dstrn cert:obtain --domain=api.example.com --email=admin@example.com --staging
dstrn cert:renew --domain=api.example.com --email=admin@example.com
dstrn cert:status
```

| コマンド | 説明 |
| -------------------------------------------- | ------------------------------------------------------------- |
| `dstrn cert <obtain\|renew\|status> [--dir=...]` | tls証明書を管理（`obtain`、`renew`、または`status`） |
| `dstrn cert:obtain --domain=... --email=... [--dir=...]` | 新しいlet's encryptのssl証明書を取得 |
| `dstrn cert:renew --domain=... --email=... [--dir=...]`  | 30日以内に期限切れになる証明書を更新 |
| `dstrn cert:status [--domain=...] [--dir=...]` | インストールされている証明書の有効性と有効期限を表示 |

<a name="アーキテクチャアナライザー"></a>

## アーキテクチャアナライザー

dframeworkにはアプリケーションの完全な依存関係グラフを構築する静的解析エンジンが含まれています。これは`graph`、`lint`、`dead`、`doctor`、`lang:lint`コマンドを駆動します。

<a name="依存関係グラフ"></a>

### 依存関係グラフ

`graph`コマンドはアプリケーションがどのように接続されているかを視覚的に表現します。すべてのhttpルートとソケットワイヤ、ディスパッチ先となるコントローラー、通過するミドルウェア、およびそれらのコントローラーが依存するモデルを表示します。

```bash
dstrn graph
```

```
 INFO  dependency graph of how the application is wired

http routes
get /users (users.index)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ index()

post /users (users.store)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ store()

socket routes
chat:message
 └─ AuthMiddleware
 └─ ChatController (Message)
     └─ onMessage()
```

機械可読な出力を行うには`--json`フラグを渡します:

```bash
dstrn graph --json
```

これにより外部の可視化ツールに適した`nodes`配列と`edges`配列を含む完全なグラフがjsonオブジェクトとして出力されます。

<a name="アーキテクチャlint"></a>

### アーキテクチャlint

`lint`コマンドはアプリケーション内の構造的な違反を検出します。

> [!IMPORTANT]
> `dstrn lint`は厳密なアーキテクチャ境界を適用します: コントローラーは他のコントローラーに依存してはならず、モデルはコントローラーに依存してはならず、ミドルウェアはコントローラーに依存してはならず、ルートはモデルに直接依存してはならず、ジョブは直接インポートすべきではありません（代わりにJob.dispatch経由でディスパッチします）。

```bash
dstrn lint
```

違反が見つかった場合それぞれが説明とともに出力されます。

```
 INFO  architecture lint

  UserController (controller) ................................  ERROR  [AdminController]
    controllers may not depend on controllers

 ERROR  found 1 architecture violation (12.30ms)
```

<a name="デッドコード検出"></a>

### デッドコード検出

`dead`コマンドは登録されたすべてのルート、ソケットワイヤ、スケジュールされたタスク（`console/Schedule.js`）から到達可能性分析を実行します。依存関係グラフを走査し参照されていないコントローラー、モデル、ジョブ、ミドルウェアを報告します。

```bash
dstrn dead
```

```
 INFO  dead code analysis

  LegacyController ...........................................  WARN  [controller]
  TempModel ..................................................  WARN  [model]

 ERROR  found 2 unreachable components (15.20ms)
```

データベーステーブルは生クエリを通じてアクセスされる可能性があるため分析から除外されます。

<a name="プロジェクトdoctor"></a>

### プロジェクトdoctor

`doctor`コマンドは3つの要素に基づいて0から100までの品質スコアを計算します:

- **アーキテクチャ違反**（1件の違反につき10点減点、最大40点）
- **デッドコード**（到達不能なコンポーネント1件につき5点減点、最大30点）
- **複雑さ**（ノードあたりの平均エッジ数が4を超える場合超過分に基づいて追加で減点）

```bash
dstrn doctor
```

```
 INFO  project architecture health

  architecture structure .....................................  OK  [no violations]
  code reachability ..........................................  WARN  [-5] (1 unreachable)
  project complexity .........................................  OK  [2.1 edges per node]

 OK  health score 95 out of 100 (10.50ms)
```

70点以上は健全、70点未満はエラーとして報告されます。

<a name="翻訳lint"></a>

### 翻訳lint

`lang:lint`コマンドは設定されているすべての言語にわたって翻訳キーを検証しビューやコード全体の参照を検証します。2つのカテゴリの問題をチェックします:

- **非対称キー**（あるロケール辞書で定義されているが別の辞書で欠落しているキー）
- **欠落した参照**（`.d`ビューまたはコントローラー内で`@t()`または`t()`により使用されているが翻訳ファイルに存在しないキー）

```bash
dstrn lang:lint
```

問題が見つかった場合コマンドは正確なファイル、行番号、欠落しているロケールを報告します:

```
 INFO  translation verification

missing translations:
  views/index.d:4 ............................................  ERROR  'ui.tagline' missing in [ja]

asymmetric translation keys:
  ja: ui.tagline .............................................  WARN  defined in [en]

 ERROR  found 2 translation issues (14.20ms)
```

すべての翻訳が完全でビューの使用状況と一致している場合は成功ステータスが返されます。

辞書内に未使用の翻訳キーが存在する場合に非ゼロの終了コードで失敗させるには`--strict`を渡します:

```bash
dstrn lang:lint --strict
```

<a name="コンパイラ"></a>

## コンパイラ

dframeworkにはフロントエンドランタイムアセットをバンドルおよび最小化するビルド時コンパイラが含まれています。

<a name="javascriptコンパイル"></a>

### javascriptコンパイル

jsコンパイラはフレームワークのコアクライアントサイドファイル（メインランタイム、ネイティブブリッジ、カスタムエレメント、ユーティリティ関数）と`public/js/components`にあるユーザー定義コンポーネントファイルを連結します。結合された出力は3回の圧縮パス、関数名のマングリング、トップレベルのマングリングにより最小化されます。結果は`storage/framework/public/js/dstrn.js`に書き込まれます。

デバッグモードが有効な`local`環境では`__DF_DEBUG__`グローバルが`true`に設定されクライアントランタイムでデバッグ専用コードパスが有効になります。本番環境ではこれらのパスはデッドコード削除中に除去されます。

<a name="cssコンパイル"></a>

### cssコンパイル

cssコンパイラはフレームワークのベーススタイル、ユーティリティクラス、アイコンフォント、コンポーネントスタイルを連結します。次に`prefix\:classname`パターンを使用して6つのブレークポイント（`sm`、`md`、`lg`、`xl`、`xxl`、`ultra`）にわたって各ユーティリティクラスのレスポンシブバリアントを生成します。`public/css/components`のユーザー定義コンポーネントcssファイルも出力に含まれます。

最終的な出力はコメントの除去、空白の圧縮、不要なセミコロンの削除、ゼロ単位サフィックスの削除によって最小化されます。結果は`storage/framework/public/css/dstrn.css`に書き込まれます。