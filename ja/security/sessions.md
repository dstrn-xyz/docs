# セッション

- [セッション](#セッション)
  - [はじめに](#はじめに)
  - [設定](#設定)
    - [セッションの有効期間](#セッションの有効期間)
    - [クッキーのオプション](#クッキーのオプション)
  - [セッションの操作](#セッションの操作)
    - [データの取得](#データの取得)
    - [データの保存](#データの保存)
    - [フラッシュデータ](#フラッシュデータ)
    - [データの削除](#データの削除)
    - [セッションIDと再生成](#セッションIDと再生成)
  - [セッションドライバー](#セッションドライバー)
    - [ステルスモード](#ステルスモード)

<a name="はじめに"></a>

## はじめに

http駆動のアプリケーションはステートレスであるためセッションは複数のリクエストにわたってユーザーに関する情報を保存する手段を提供します。dframeworkは基盤となるストレージドライバーに関係なくセッションデータを操作するためのエレガントでグローバルに利用可能な`Session`ファサードを提供します。

<a name="設定"></a>

## 設定

すべてのセッション設定は`config/session.js`でのみ定義されます。セッションドライバーは`driver`（`memory`、`database`、または`stealth`）によって決定されセッションクッキー名は`cookie`（デフォルトは`sid`）によって定義されます。

<a name="セッションの有効期間"></a>

### セッションの有効期間

セッションの有効期間はセッションが期限切れになるまでにアクティブな状態を維持する時間を決定します。`lifetime`を使用して分単位で期間を設定できます（デフォルトは120分）。

```javascript
// config/session.js
import { Env } from 'dframework';

export default {
  driver: Env.value('SESSION_DRIVER', 'memory'),
  cookie: Env.value('SESSION_COOKIE', 'sid'),
  lifetime: Env.value('SESSION_LIFETIME', 120),
  domain: Env.value('SESSION_DOMAIN', null),
  path: Env.value('SESSION_PATH', '/'),
  same_site: Env.value('SESSION_SAME_SITE', 'Lax'),
  http_only: Env.value('SESSION_HTTP_ONLY', true),
  secure: Env.value('SESSION_SECURE', null),
};
```

<a name="クッキーのオプション"></a>

### クッキーのオプション

`config/session.js`でセッションクッキーのセキュリティ属性を設定できます。

- `same_site`: クロスサイトクッキー送信を制御します（デフォルトは`'Lax'`、指定可能な値: `'Lax'`、`'Strict'`、`'None'`）
- `secure`: https経由でのクッキー送信を強制します（デフォルトは`null`でありアプリケーション環境がlocal以外の場合はセキュアクッキーを自動的に有効化しローカル開発環境では無効化します、指定可能な値: `true`、`false`、`null`）
- `domain`: クッキーのドメインスコープを定義します（デフォルトは`null`でありサブドメインなしで現在のホストオリジンに厳格に制限されます、指定可能な値: `'.example.com'`などのドメイン文字列または`null`）
- `path`: クッキーのパススコープを定義します（デフォルトは`'/'`、指定可能な値: `'/'`などのパス文字列）
- `http_only`: クライアントサイドスクリプトによるセッションクッキーへのアクセスを防止します（デフォルトは`true`、指定可能な値: `true`、`false`）

<a name="セッションの操作"></a>

## セッションの操作

<a name="データの取得"></a>

### データの取得

セッションからアイテムを取得するにはグローバルの`Session`ファサードの`get`メソッドを使用します。第2引数としてデフォルト値を渡すことができ指定されたキーが存在しない場合にその値が返されます。

```javascript
// retrieve a specific key
const value = await Session.get('key');

// retrieve a key with a default fallback
const name = await Session.get('name', 'guest');
```

<a name="データの保存"></a>

### データの保存

セッションにデータを保存するには`set`メソッドを使用します。

```javascript
await Session.set('key', 'value');
```

データを恒久的に保存する必要がある場合（10年間の長期有効クッキーを使用）は`permanent`メソッドを使用します。キーと値のペアまたはオブジェクトを渡すことができます。

```javascript
await Session.permanent('role', 'admin');
await Session.permanent({ role: 'admin', accepted_terms: true });
```

<a name="フラッシュデータ"></a>

### フラッシュデータ

次のリクエストのためだけにアイテムをセッションに保存したい場合があります。その場合は`flash`メソッドを使用できます。このメソッドを使用して保存されたデータは即座に利用可能となりその直後のhttpリクエストの間保持された後自動的に削除されます。これは短命なステータスメッセージに非常に役立ちます。

```javascript
await Session.flash('status', 'profile updated successfully');
```

<a name="データの削除"></a>

### データの削除

セッションからデータの一部を削除するには`forget`メソッドを使用して特定のキーを渡します。引数なしで`forget`を呼び出した場合セッション全体が破棄されクッキーが無効化されます。

```javascript
// forget a single key
await Session.forget('key');

// destroy the entire session
await Session.forget();

// remove all data from the current session including authentication data
await Session.flush();
```

> [!NOTE]
> `Session.flush()`は現在のセッションに保存されているすべてのデータ（認証情報やカスタムペイロードキーを含む）を削除しセッションを無効化します。セッションを維持したまま単一のキーを選択して削除するには`Session.forget(key)`を呼び出します。

<a name="セッションIDと再生成"></a>

### セッションIDと再生成

アクティブなセッション識別子を取得したりカスタム識別子を設定したりするには`id`メソッドを使用します。

```javascript
// retrieve the current session id
const sessionId = Session.id();
```

ユーザー権限が変更された際（ログインやパスワードリセット時など）のセッション固定化攻撃を防ぐには`regenerate`メソッドを使用します。デフォルトでは既存のセッションデータを保持したまま新しいセッションIDとクッキーを発行します。すべてのデータを消去して新しいセッションを発行するには`false`を渡します。

```javascript
// regenerate session id while keeping existing session data
await Session.regenerate();

// regenerate session id and clear all existing session data
await Session.regenerate(false);
```

<a name="セッションドライバー"></a>

## セッションドライバー

dframeworkはセッションストレージの複雑さをシンプルなドライバーの背後に抽象化しています。

- **memory**: セッションをramに保存します。高速ですがサーバー再起動時に失われます。ローカル開発に適しています。
- **database**: 専用のデータベーステーブルにセッションを保存します。堅牢であり複数サーバーインスタンス間で拡張可能です。
- **stealth**: セッションを暗号化されたクッキーとして完全にクライアントサイドに保存します。サーバーサイドのストレージやデータベースクエリを一切必要としません。

<a name="ステルスモード"></a>

### ステルスモード

dframeworkにはプライバシーとゼロステートアーキテクチャのために設計された`stealth`セッションドライバーが含まれています。

ステルスモードが有効な場合（`SESSION_DRIVER=stealth`）セッションストレージはデータベースクエリゼロ、redisネットワーク往復ゼロ、サーバーメモリ消費ゼロで動作します。

```javascript
// config/session.js
import { Env } from 'dframework';

export default {
  driver: Env.value('SESSION_DRIVER', 'stealth'),
  cookie: Env.value('SESSION_COOKIE', 'sid'),
  lifetime: Env.value('SESSION_LIFETIME', 120),
  same_site: 'Lax',
  http_only: true,
  secure: null
};
```

#### ステルスセッションの仕組み

セッションペイロード全体はアプリケーション秘密鍵（`APP_KEY`）を使用してaes 256 gcmでシリアライズおよび暗号化されます。結果として得られる暗号化トークン文字列がセッション識別子として機能し設定されたhttp onlyクッキーでブラウザに返送されます。

受信リクエスト時フレームワークはクッキーを復号します。クッキーが攻撃者によって改ざんまたは変更されている場合復号は安全に失敗し`app.debug`が有効な場合にデバッグ警告をログに記録して空のゲストセッションを初期化します。

暗号化されたペイロードの内部には有効期限タイムスタンプが埋め込まれています。セッション状態を保存する際ドライバーは有効期限ウィンドウを自動的に更新するか将来の有効期限日（永続セッションなど）を保持します。期限切れのセッションはnullを返し破棄されます。

セッション全体がhttpクッキーヘッダー内に保存されるため保存するデータは最小限に抑える必要があります（webブラウザのクッキーサイズ制限内に安全に収めるため厳格に4kb未満にする必要があります）。ステルスモードはユーザー識別子、認証トークン、最小限の設定情報に最適ですが大きな配列やかさばるオブジェクトの保存には使用しないでください。

サーバー上にはセッション行もファイルも存在しないためマイグレーションするデータベーステーブルはなく定期的なガベージコレクションコマンドも不要です。
