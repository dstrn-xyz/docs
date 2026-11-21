# ストレージ

- [ストレージ](#ストレージ)
  - [はじめに](#はじめに)
  - [設定](#設定)
  - [ディスクインスタンスの取得](#ディスクインスタンスの取得)
  - [ファイルの保存](#ファイルの保存)
    - [ファイルアップロード](#ファイルアップロード)
  - [ファイルの取得](#ファイルの取得)
  - [ファイルの削除](#ファイルの削除)
  - [ファイル管理](#ファイル管理)
  - [暗号化](#暗号化)
  - [ディスク容量の保護](#ディスク容量の保護)

<a name="はじめに"></a>

## はじめに

dframeworkは`Storage`ファサードを通じてファイルシステム抽象化を提供します。設定された複数のディスクにわたってファイルを簡単に読み書きおよび管理できます。ストレージサービスはパストラバーサルの防止や公開アセットのシンボリックリンク作成、並行操作キューイング、ディスク容量の検証を自動的に管理します。

<a name="設定"></a>

## 設定

ストレージ設定は`config/storage.js`の`storage.disks`設定名前空間内で定義します。一般的な設定にはプライベートファイル用の`local`ディスク、公開アセット用の`public`ディスク、暗号化ファイル用の`secure`ディスクが含まれます。

```javascript
import { Env } from 'dframework';

export default {
  disks: {
    local: {
      root: './storage/local',
    },
    public: {
      root: './storage/public',
    },
    secure: {
      root: './storage/secure',
      encryptionKey: Env.value('APP_KEY'),
    },
  },
};
```

dframeworkは起動時に`public/storage`から`storage/public`へのシンボリックリンクを自動作成するため`public`ディスクに保存されたファイルはブラウザへ直接配信できます。

<a name="ディスクインスタンスの取得"></a>

## ディスクインスタンスの取得

ディスクインスタンスを取得する標準的な記述は`Storage.disk(name)`の呼び出しです。ディスク名を省略すると`config/storage.js`で設定されたデフォルトディスク（未設定時は`public`）に解決されます。

利便性のために`Storage`ファサードはデフォルトディスクへの直接プロキシ操作（`Storage.put()`、`Storage.get()`、`Storage.delete()`など）も提供します。

```javascript
import { Storage } from 'dframework';

await Storage.disk().put('notes.txt', 'hello');

await Storage.put('notes.txt', 'hello');

const disk = Storage.disk('local');
```

<a name="ファイルの保存"></a>

## ファイルの保存

ディスクにファイルを書き込むには`put`メソッドを使用します。第1引数に相対ファイルパス、第2引数にファイル内容（文字列またはバッファ）を渡します。必要なディレクトリは自動作成されます。

```javascript
await Storage.disk().put('avatars/1.jpg', imageBuffer);
await Storage.disk('local').put('exports/report.csv', 'id,name\n1,tarou');
```

<a name="ファイルアップロード"></a>

### ファイルアップロード

`put`メソッドはhttpアップロード時に生成されるマルチパートファイルオブジェクトに対応しています。アップロードされたファイルオブジェクトを直接`put`に渡すことで手動でのバッファ展開なしにストリーミング書き込みが実行されます。

```javascript
const path = await Storage.disk().put('avatars/user.jpg', req.file('avatar'));
```

<a name="ファイルの取得"></a>

## ファイルの取得

ファイル内容を読み取るには`get`メソッドを使用します。デフォルトでは生の`Buffer`を返します。文字列として取得したい場合は第2引数に`utf8`などの文字エンコーディングを指定します。

```javascript
const raw = await Storage.disk('local').get('documents/contract.pdf');
const text = await Storage.disk('local').get('reports/summary.txt', 'utf8');
```

ファイルが存在するかどうかは`exists`メソッドで確認できます。

```javascript
if (await Storage.disk().exists('avatars/1.jpg')) {
}
```

ディスク上の絶対パスが必要な場合は`path`メソッドを使用します。

```javascript
const absolutePath = Storage.disk('local').path('exports/report.csv');
```

<a name="ファイルの削除"></a>

## ファイルの削除

ファイルを削除するにはパスを`delete`メソッドに渡します。ファイルが削除された場合は`true`、存在しなかった場合は`false`を返します。

```javascript
await Storage.disk().delete('avatars/1.jpg');
```

<a name="ファイル管理"></a>

## ファイル管理

同一ディスク内でのファイル移動や複製を行うメソッドも用意されています。

```javascript
await Storage.disk().move('temp/draft.txt', 'published/final.txt');

await Storage.disk().copy('templates/invoice.pdf', 'invoices/123.pdf');
```

<a name="暗号化"></a>

## 暗号化

`encryptionKey`が設定されたディスクは書き込み時に自動暗号化され読み取り時に自動復号されます。アルゴリズムはaes-256-gcmです。鍵は提供された文字列からsha256経由で導出されます。暗号化キーはファイル内には保存されず環境設定（`APP_KEY`）内でのみ保持されます。apiは非暗号化ディスクと同一です。

ディスク上のフォーマットは12バイトivと16バイト認証タグと暗号文で構成されます。認証タグにより改ざん検知が行われファイルが変更されていた場合は読み取りが失敗します。

```javascript
await Storage.disk('secure').put('keys/api-key.txt', 'my-secret-value');

const value = await Storage.disk('secure').get('keys/api-key.txt', 'utf8');
```

<a name="ディスク容量の保護"></a>

## ディスク容量の保護

ディスク容量枯渇によるアプリケーション停止を防ぐためストレージサービスは書き込み前に容量の健全性チェックを実行します。

> [!NOTE]
> `put()`による書き込み前にストレージサービスは対象ディスクにデータサイズに加えて100メガバイトの安全マージンがあるかを検証します。容量が不足している場合書き込み処理は安全に中断されます。
