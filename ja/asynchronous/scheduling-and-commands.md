# スケジューリングとコマンド

- [スケジューリングとコマンド](#スケジューリングとコマンド)
  - [はじめに](#はじめに)
  - [スケジュールの定義](#スケジュールの定義)
    - [スケジュールファイル](#スケジュールファイル)
    - [コマンドのスケジューリング](#コマンドのスケジューリング)
    - [クロージャのスケジューリング](#クロージャのスケジューリング)
  - [実行頻度オプション](#実行頻度オプション)
    - [プリセット間隔](#プリセット間隔)
    - [カスタム間隔](#カスタム間隔)
  - [コマンドの作成](#コマンドの作成)
    - [コマンドクラス](#コマンドクラス)
    - [コマンドの生成](#コマンドの生成)
    - [コマンドの実行](#コマンドの実行)
  - [スケジューラーの仕組み](#スケジューラーの仕組み)
    - [自動起動](#自動起動)
    - [実行と再スケジュール](#実行と再スケジュール)
    - [手動実行](#手動実行)
    - [スケジュールタスクの確認](#スケジュールタスクの確認)
  - [エラーハンドリング](#エラーハンドリング)

<a name="はじめに"></a>

## はじめに

dframeworkは定期的なバックグラウンドタスクをアプリケーションコード内で直接定義できる組み込みタスクスケジューラーを備えています。サーバー上でシステムレベルのcronジョブを設定する代わりに単一のファイルでスケジュールを定義するだけでフレームワークが残りの処理を管理します。タスクにはコマンドクラスへの参照またはインライン関数を指定できます。

スケジューラーはアプリケーションの起動時に自動的に開始されプロセスの生存期間中動作し続けます。各タスクは分離されたコンテキストでメインスレッド内で同期的に実行され完了後に自動的に再スケジュールされます。

<a name="スケジュールの定義"></a>

## スケジュールの定義

<a name="スケジュールファイル"></a>

### スケジュールファイル

スケジュールはプロジェクトルートの`console/Schedule.js`で定義します。このファイルは`Scheduler`インスタンスを受け取るデフォルト関数をエクスポートします。タスクを登録するにはスケジューラーの`command()`メソッドを使用し実行頻度メソッドをチェーンします。

```javascript
export default function (scheduler) {
  scheduler.command('CleanupExpiredSessions').daily();
  scheduler.command('SyncExternalData').everyFifteenMinutes();
  scheduler.command('GenerateSitemaps').weekly();
}
```

フレームワークは起動時にこのファイルを自動的に検出し定義されたスケジュールの実行を開始します。

<a name="コマンドのスケジューリング"></a>

### コマンドのスケジューリング

`scheduler.command()`に文字列を渡すとスケジューラーは`console/commands`ディレクトリ内のコマンドファイルとして解決し`${name}.js`と`${name}Command.js`の両方を検索します。このファイルは`handle()`メソッドを持つクラスをデフォルトエクスポートする必要があります。

```javascript
scheduler.command('PruneOldLogs').daily();
scheduler.command('Report').args(['--format=pdf', 'sales']).daily();
```

これにより`console/commands/PruneOldLogs.js`または`console/commands/PruneOldLogsCommand.js`が解決されクラスがインスタンス化され指定された間隔で`handle()`メソッドが呼び出されます。`.args(argsArray)`をチェーンすることで引数とオプションをコマンドインスタンスに転送できます。

<a name="クロージャのスケジューリング"></a>

### クロージャのスケジューリング

個別のコマンドファイルを必要としない単純なタスクの場合はインライン関数を直接渡すことができます。

```javascript
export default function (scheduler) {
  scheduler.command(async () => {
    const count = await Session.where('expires_at', '<', new Date()).delete();
    Log.info(`pruned ${count} expired sessions`);
  }).everyThirtyMinutes();
}
```

<a name="実行頻度オプション"></a>

## 実行頻度オプション

<a name="プリセット間隔"></a>

### プリセット間隔

スケジューラーは`command()`の後にチェーンできる直感的なプリセット間隔メソッドのセットを提供します。

| メソッド | 間隔 |
| --- | --- |
| `everyMinute()` | 60秒 |
| `everyFiveMinutes()` | 5分 |
| `everyTenMinutes()` | 10分 |
| `everyFifteenMinutes()` | 15分 |
| `everyThirtyMinutes()` | 30分 |
| `hourly()` | 1時間 |
| `daily()` | 24時間 |
| `weekly()` | 7日 |

```javascript
scheduler.command('HeartbeatCheck').everyMinute();
scheduler.command('CacheWarmer').hourly();
scheduler.command('WeeklyDigest').weekly();
```

<a name="カスタム間隔"></a>

### カスタム間隔

プリセットでカバーされていない間隔についてはミリ秒単位の期間を指定して`every()`メソッドを使用します。

```javascript
scheduler.command('PollExternalApi').every(45000);
scheduler.command('RotateApiKeys').every(12 * 60 * 60 * 1000);
```

nodeの32ビット整数タイムアウト制限のため間隔は2147483647ミリ秒（約24.8日）を超えることはできません。この期間を超える間隔（月次や四半期ごとのジョブなど）については`dstrn run <command>`を使用して必要に応じてコマンドを実行するシステムcronジョブを設定してください。

<a name="コマンドの作成"></a>

## コマンドの作成

<a name="コマンドクラス"></a>

### コマンドクラス

コマンドは`console/commands`ディレクトリに保存されるクラスです。`async handle(args, options)`メソッドを持つクラスをデフォルトエクスポートする必要があります。`handle`メソッドにはコマンド実行時に実行されるロジックが含まれます。

```javascript
export default class PruneOldLogsCommand {
  async handle(args, options) {
    const cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - 30);
    await DB.table('logs').where('created_at', '<', cutoff).delete();
    Log.info('old logs pruned');
  }
}
```

スケジュールされたコマンドインスタンスは引数ヘルパー（`this.option()`、`this.hasOption()`、`this.argument()`、`this.options`、`this.arguments`、`this.rawArgs`）を提供します。自動スケジューラーによってトリガーされたコマンドは空の引数（`handle([], {})`）で実行され`dstrn run`による手動実行では引数とオプションが転送されます（`handle(rawArgs, options)`）。

<a name="コマンドの生成"></a>

### コマンドの生成

cliを使用して新しいコマンドファイルをスキャフォールディングできます。

```bash
dstrn make:command PruneOldLogs
```

これにより適切な定型構造を持つ`console/commands/PruneOldLogsCommand.js`が作成されます（ジェネレーターは末尾に`Command`サフィックスがない場合自動的に追加します）。

<a name="コマンドの実行"></a>

### コマンドの実行

`run` cliコマンドを使用してスケジュールされた任意のコマンドをオンデマンドで実行できます。

```bash
dstrn run PruneOldLogs
```

`dstrn run`経由で手動実行する場合コマンド名の後に渡された追加の引数とオプションは`handle(args, options)`に転送され`this.option()`および`this.argument()`からアクセスできます。自動スケジューラーによってトリガーされた場合は`handle([], {})`が空の引数で実行されます。dframeworkは完了時に経過した実行時間を報告します。

<a name="スケジューラーの仕組み"></a>

## スケジューラーの仕組み

<a name="自動起動"></a>

### 自動起動

アプリケーションの起動時フレームワークは`console/Schedule.js`ファイルの存在を確認します。存在する場合フレームワークはそれをインポートし新しい`Scheduler`インスタンスを渡しすべてのタスクが登録された後に`scheduler.start()`を呼び出します。登録された各タスクは設定された間隔の経過後に実行されるようスケジュールされその後継続的に再スケジュールされます。

<a name="実行と再スケジュール"></a>

### 実行と再スケジュール

タスクの間隔が経過するとスケジューラーはタスクのハンドラー（コマンドクラスまたはインライン関数）を呼び出します。デバッグモードではスケジューラーは実行開始と完了時の所要時間をログに記録します（`[scheduler] running scheduled task: <task>`、`[scheduler] scheduled task <task> completed in <duration>ms`）。タスク完了後スケジューラーは設定された間隔から経過した実行時間を差し引くことで次回の実行時間を計算します。これによりタスクの完了に大幅な時間がかかった場合でも実行間隔の一貫性が保たれます。次回の実行は`setTimeout`を使用してスケジュールされます。

各タスクは`lastRun`タイムスタンプを追跡します。スケジューラーはこれを使用して次回実行までの適切な遅延を算出します。

<a name="手動実行"></a>

### 手動実行

`scheduler.run()`を使用して登録された任意のスケジュールタスクを名前でプログラムからトリガーできます。

```javascript
await app.scheduler.run('CleanupExpiredSessions');
```

これによりスケジュールされた定期タイマーを変更することなくオンデマンドでタスクを即座に1回実行します。

<a name="スケジュールタスクの確認"></a>

### スケジュールタスクの確認

登録済みのタスク、実行間隔、次回実行予定時刻、前回の終了ステータスをcliから確認するには:

```bash
dstrn schedule:list
dstrn schedule:list --json
```

<a name="エラーハンドリング"></a>

## エラーハンドリング

> [!NOTE]
> スケジュールされたタスクによってスローされたエラーはログに記録され分離されます。失敗したタスクは他のタスクに影響を与えたりサーバーを中断させたりすることなく自動的に再スケジュールされます。

```
[scheduler] scheduled task PruneOldLogs failed after 12ms: ER_NO_SUCH_TABLE: Table 'logs' doesn't exist
```

スケジュールフェーズ中にタスク自体が予期しないエラーを引き起こした場合個別のcatchがそれをログに記録しエラーがメインプロセスに伝播するのを防ぎます。
