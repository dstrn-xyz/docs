# ジョブとキュー

- [ジョブとキュー](#ジョブとキュー)
  - [はじめに](#はじめに)
  - [ジョブの作成](#ジョブの作成)
    - [ジョブクラス](#ジョブクラス)
    - [ジョブ設定](#ジョブ設定)
    - [ジョブの生成](#ジョブの生成)
  - [ジョブのディスパッチ](#ジョブのディスパッチ)
    - [コントローラーから](#コントローラーから)
    - [どこからでも](#どこからでも)
    - [コマンドラインから](#コマンドラインから)
    - [ペイロードデータ](#ペイロードデータ)
  - [Jobファサード](#Jobファサード)
  - [ワーカープールの仕組み](#ワーカープールの仕組み)
    - [ワーカーの分離](#ワーカーの分離)
    - [同時実行制限](#同時実行制限)
    - [自動キューイングと永続ワーカープール](#自動キューイングと永続ワーカープール)
  - [ジョブのライフサイクル](#ジョブのライフサイクル)
    - [ブートストラップ](#ブートストラップ)
    - [データベースアクセス](#データベースアクセス)
    - [ジョブからのブロードキャスト](#ジョブからのブロードキャスト)
    - [エラーハンドリング](#エラーハンドリング)
  - [設定](#設定)

<a name="はじめに"></a>

## はじめに

dframeworkは分離されたワーカースレッドでジョブを実行するネイティブのバックグラウンドジョブキューを備過しています。ジョブはアプリケーションコードからディスパッチされメインhttpプロセスをブロックすることなく非同期に実行されます。これはメール送信、画像処理、外部api同期などユーザーへの応答を遅延させるべきでないタスクに最適です。

> [!NOTE]
> キューシステムはredisや個別のキューサーバーのような外部依存関係を一切必要としません。組み込みの`worker_threads`モジュールを使用してnode.jsランタイム内で完全に動作します。

<a name="ジョブの作成"></a>

## ジョブの作成

<a name="ジョブクラス"></a>

### ジョブクラス

ジョブはプロジェクト直下の`jobs`ディレクトリに保存されるクラスです。すべてのジョブは`handle(payload)`メソッドを持つクラスをデフォルトエクスポートする必要があります。`handle`メソッドはジョブディスパッチ時に渡されたペイロードを受け取り実際に実行する処理を含みます。

```javascript
export default class SendWelcomeEmailJob {
  async handle(payload) {
    const user = await User.find(payload.userId);
    await Mailer.send(user.email, 'welcome', { name: user.name });
  }
}
```

`handle`メソッドは非同期にできます。ワーカーは返されたpromiseが解決されるのを待ってから完了を報告します。

<a name="ジョブ設定"></a>

### ジョブ設定

ジョブはクラスの静的プロパティとしてデフォルトの実行オプションを直接定義できます。これらは呼び出し元で上書きされない限りジョブのディスパッチ時に自動的に適用されます。

```javascript
export default class ProcessVideoJob {
  static timeout = 30000;
  static tries = 3;
  static backoff = 1000;

  async handle(payload) {
    await this.transcode(payload.videoId);
  }
}
```

| プロパティ | 型 | デフォルト値 | 説明 |
| --- | --- | --- | --- |
| `timeout` | `number` | `60000` | ミリ秒単位の実行タイムアウト（このタイムアウトを超過したワーカーは終了します） |
| `tries` | `number` | `1` | ジョブが失敗としてマークされるまでの最大試行回数 |
| `backoff` | `number` | `1000` | 再試行間のミリ秒単位の指数バックオフ基本遅延 |

<a name="ジョブの生成"></a>

### ジョブの生成

cliを使用して新しいジョブファイルをスキャフォールディングできます。

```bash
dstrn make:job SendWelcomeEmailJob
```

これにより適切な定型構造を持つ`jobs/SendWelcomeEmailJob.js`ファイルが作成されます。

<a name="ジョブのディスパッチ"></a>

## ジョブのディスパッチ

<a name="コントローラーから"></a>

### コントローラーから

ジョブをディスパッチする最も一般的な方法はユーザーリクエストを処理した後にコントローラーから行うことです。

```javascript
export default class RegistrationController {
  async register(req, res) {
    const user = await User.create(req.body);
    Job.dispatch('SendWelcomeEmailJob', { userId: user.id });
    return redirect('/dashboard');
  }
}
```

`Job.dispatch()`はノンブロッキングです。ジョブがバックグラウンドワーカースレッドで実行される間即座にコントローラーに制御を返します。

<a name="どこからでも"></a>

### どこからでも

`Job`ファサードはグローバルに利用可能です。コントローラー、ミドルウェア、ソケットイベントハンドラー、他のジョブ、スケジュールされたコマンドからジョブをディスパッチすることもフレームワークルートから直接インポートすることもできます。

```javascript
import { Job } from 'dframework';

Job.dispatch('SyncInventoryJob', { warehouseId: 3 });
```

またはすべてのランタイムコンテキストで自動的に利用可能なグローバル`Job`ファサードを使用します。

```javascript
Job.dispatch('GenerateReportJob', { month: 'june', year: 2026 });
```

<a name="コマンドラインから"></a>

### コマンドラインから

`dstrn dispatch`コマンドを使用してコマンドラインからジョブをディスパッチすることもできます。これは開発中のジョブテストや手動での単発タスク実行に役立ちます。

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}'
```

`--payload`フラグはパースされてジョブの`handle`メソッドに渡されるjson文字列を受け取ります。省略された場合ジョブは空のオブジェクトを受け取ります。このコマンドはワーカースレッドと同様にデータベース接続やすべてのフレームワークグローバルを含む完全なアプリケーションコンテキストをブートストラップします。完了時に実行時間が報告されます。

```bash
dstrn dispatch CleanupExpiredSessionsJob
```

シェルのタブ補完を容易にするため`.js`で終わるファイル名も受け付けます。

```bash
dstrn dispatch CleanupExpiredSessionsJob.js
```

<a name="ペイロードデータ"></a>

### ペイロードデータ

`Job.dispatch()`の第2引数はjsonにシリアライズされてジョブの`handle`メソッドに渡されるプレーンなjavascriptオブジェクトです。ペイロードは小さくシリアライズ可能な状態を維持してください。完全なオブジェクトではなく識別子を渡すようにします。

```javascript
Job.dispatch('ProcessImageJob', {
  imageId: upload.id,
  operations: ['resize', 'compress'],
  quality: 85
});
```

<a name="Jobファサード"></a>

## Jobファサード

グローバルの`Job`はアプリケーションの`Queue`インスタンスに処理を委譲するプロキシファサードです。アプリケーションの起動時に自動的に登録されグローバルスコープで利用可能です。

```javascript
Job.dispatch('JobName', { key: 'value' });
```

`Job.dispatch()`は実行オプションを指定するオプションの第3引数を受け付けます（ジョブクラスで定義された静的プロパティを上書きします）。

```javascript
Job.dispatch('ProcessVideoJob', { videoId: 42 }, {
  timeout: 30000,
  tries: 3,
  backoff: 1000
});
```

| オプション | 型 | デフォルト値 | 説明 |
| --- | --- | --- | --- |
| `timeout` | `number` | `60000` | ミリ秒単位の実行タイムアウト（このタイムアウトを超過したワーカーは終了します） |
| `tries` | `number` | `1` | ジョブが失敗としてマークされるまでの最大試行回数 |
| `backoff` | `number` | `1000` | 再試行間のミリ秒単位の指数バックオフ基本遅延 |

アプリケーションが初期化される前に`Job`ファサードを使用しようとすると詳細なエラーがスローされます。

<a name="ワーカープールの仕組み"></a>

## ワーカープールの仕組み

<a name="ワーカーの分離"></a>

### ワーカーの分離

ディスパッチされた各ジョブはnode `worker_threads`を使用した専用の`Worker`スレッドで実行されます。ワーカースレッドはhttpサーバーを起動することなくデータベース接続、設定の読み込み、すべてのフレームワークグローバルを含む最小限のアプリケーションインスタンスを初期化します。これによりジョブコードは`DB`ファサード、`Config`、モデルクラス、すべてのヘルパー関数を含むコントローラーとまったく同じapiを使用できます。

<a name="同時実行制限"></a>

### 同時実行制限

キューは同時実行ワーカーの最大数を強制します。デフォルトでは最大4つのジョブを同時に実行できます。新しいジョブがディスパッチされたときにすべてのワーカースロットが占有されている場合ジョブはメモリ内キューに配置されワーカーが利用可能になり次第実行されます。

最大ワーカー数は`queue.maxWorkers`設定キーによって設定可能です。

<a name="自動キューイングと永続ワーカープール"></a>

### 自動キューイングと永続ワーカープール

ワーカープールが上限に達した場合追加でディスパッチされたジョブは内部キューに自動的にバッファリングされます。dframeworkのワーカーは永続的なワーカースレッドプール内で動作します。

1. オンデマンド実行: ワーカースレッドは`Job.dispatch()`によってジョブがディスパッチされたときに遅延生成されます。バックグラウンドジョブをディスパッチしないアプリケーションではワーカーオーバーヘッドが一切発生しません。
2. 永続実行: ジョブが存在する場合ワーカースレッドはアプリケーション環境とデータベースプールを一度ブートしアイドルプールで生存し続けます。後続のジョブはスレッド生成やアイソレートの起動遅延なしにウォームワーカーによって即座に処理されます。
3. 自動デキュー: 各ワーカーはジョブを終了すると内部キューから次のバッファリングされたジョブを取得するか今後のディスパッチに備えてアイドルプールに戻ります。
4. 障害回復: ワーカースレッド内で処理されない致命的なエラーやプロセスクラッシュが発生した場合ワーカーはクリーンアップを行って終了し保留中のタスクの処理を継続するために代替ワーカーが生成されます。

<a name="ジョブのライフサイクル"></a>

## ジョブのライフサイクル

<a name="ブートストラップ"></a>

### ブートストラップ

永続ワーカースレッドが起動すると以下の手順が実行されます。

1. プロジェクトから環境変数と設定ファイルを読み込みます
2. データベース接続とグローバルファサードを持つアプリケーションインスタンスを初期化します
3. メインプロセスに準備完了を通知しディスパッチされたジョブを待機します
4. ジョブを受け取ると`jobs`ディレクトリからジョブクラスをインポートします
5. ジョブクラスをインスタンス化し`handle`メソッドを検証します
6. シリアライズされたペイロードを渡して`handle(payload)`を呼び出します
7. 実行時間を添えて成功または失敗をメインスレッドに報告します
8. 後続のジョブを即座に受け付けるためにアイドルワーカープールに戻ります

<a name="データベースアクセス"></a>

### データベースアクセス

ジョブはコントローラーで利用可能なものと同じ`DB`ファサードとモデルクラスを通じてデータベースへの完全なアクセス権を持ちます。各ワーカースレッドは初期化中にデータベース接続プールを確立します。プールはワーカーの生存期間中ジョブの実行をまたいでオープンな状態を維持しワーカーが終了するかキューが空になったときに安全に破棄されます。

```javascript
export default class CleanupExpiredSessionsJob {
  async handle(payload) {
    const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    await DB.table('sessions').where('updated_at', '<', cutoff).delete();
  }
}
```

<a name="ジョブからのブロードキャスト"></a>

### ジョブからのブロードキャスト

ジョブはグローバルな`Socket`ファサードを使用してリアルタイムwebsocketイベントをディスパッチし接続されているクライアントのコンポーネント状態を更新できます。ジョブは専用のワーカースレッド内で実行されるため`Socket.broadcast()`および`Socket.setState()`は内部ワーカーチャネル経由でメインプロセスにメッセージを自動的にリレーしメインプロセスが接続されているブラウザにブロードキャストします。

```javascript
export default class GenerateReportJob {
  async handle(payload) {
    Socket.broadcast('report:progress', { percent: 25 });

    const report = await this.generate(payload.reportId);
    Socket.broadcast('report:progress', { percent: 100 });

    Socket.broadcast('report:ready', {
      reportId: report.id,
      downloadUrl: `/reports/${report.id}/download`
    });

    Socket.setState('#report-banner', {
      status: 'complete',
      reportId: report.id
    });
  }
}
```

外部メッセージブローカー（redisなど）は不要です。ワーカースレッドはプライマリサーバーインスタンスと直接通信します。

<a name="エラーハンドリング"></a>

### エラーハンドリング

ジョブの`handle`メソッドがエラーをスローした場合エラー名、メッセージ、スタックトレース、実行時間を含む構造化エラーメッセージがメインスレッドに送り返されアプリケーションロガーによって記録されます。ワーカースレッドは終了する前にデータベース接続を閉じるためのクリーンアップ関数を実行します。

```
[JOB] [SendWelcomeEmailJob] failed after 1847ms:
Error: SMTP connection refused
    at Mailer.send (mailer/Mailer.js:42)
    at SendWelcomeEmailJob.handle (jobs/SendWelcomeEmailJob.js:12)
```

ワーカースレッド自体がクラッシュした場合（例えばメモリ不足状態によるものなど）メインスレッドは`error`イベントを受け取りクラッシュをログに記録します。ワーカースロットが解放されキュー内の次のジョブが通常どおり処理されます。ワーカープロセスが終了するとデータベース接続は自動的に解放されます。

<a name="設定"></a>

## 設定

キューシステムは`app.queue`設定名前空間から設定を読み込みます。

| 設定キー | 型 | デフォルト値 | 説明 |
| --- | --- | --- | --- |
| `app.queue.maxWorkers` | `number` | `4` | 同時実行ワーカースレッドの最大数 |
| `app.queue.timeout` | `number` | `60000` | ミリ秒単位のデフォルトジョブ実行タイムアウト |
| `app.queue.bootstrapTimeout` | `number` | `30000` | 新規起動したワーカーのブートストラップ完了を待機する最大ミリ秒数 |
| `app.queue.shutdownTimeout` | `number` | `30000` | グレースフルシャットダウン時に実行中ジョブの完了を待機する最大ミリ秒数 |

```javascript
// config/app.js
export default {
  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
    timeout: Env.value('QUEUE_TIMEOUT', 60000),
    bootstrapTimeout: Env.value('QUEUE_BOOTSTRAP_TIMEOUT', 30000),
    shutdownTimeout: Env.value('QUEUE_SHUTDOWN_TIMEOUT', 30000),
  },
};
```
