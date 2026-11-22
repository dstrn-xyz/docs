# WebSocket

- [WebSocket](#WebSocket)
  - [はじめに](#はじめに)
  - [仕組み](#仕組み)
  - [ソケットイベントの定義](#ソケットイベントの定義)
    - [イベントハンドラー](#イベントハンドラー)
    - [インラインハンドラー](#インラインハンドラー)
    - [コントローラーハンドラー](#コントローラーハンドラー)
  - [ミドルウェア](#ミドルウェア)
    - [インラインミドルウェア](#インラインミドルウェア)
    - [グループ化ミドルウェア](#グループ化ミドルウェア)
  - [イベントへの応答](#イベントへの応答)
    - [HTMLのレンダリング](#HTMLのレンダリング)
    - [JSONの送信](#JSONの送信)
    - [リダイレクト](#リダイレクト)
    - [処理の中断](#処理の中断)
    - [クッキー](#クッキー)
  - [ターゲット制限](#ターゲット制限)
  - [ブロードキャスト](#ブロードキャスト)
  - [ターゲット配信](#ターゲット配信)
    - [特定ユーザーへの送信](#特定ユーザーへの送信)
    - [複数ユーザーへの送信](#複数ユーザーへの送信)
    - [セッションへの送信](#セッションへの送信)
    - [ダイレクトソケットターゲット](#ダイレクトソケットターゲット)
    - [マルチガードターゲット](#マルチガードターゲット)
    - [ユーザー接続の取り消し](#ユーザー接続の取り消し)
  - [Socketファサード](#Socketファサード)
  - [クライアントサイドでの使用](#クライアントサイドでの使用)
    - [接続](#接続)
    - [イベントの送信](#イベントの送信)
    - [イベントの待機](#イベントの待機)
  - [オリジン保護](#オリジン保護)
  - [デバッグモード](#デバッグモード)

<a name="はじめに"></a>

## はじめに

dframeworkは完全に統合されたソケットレイヤーを備えています。ミドルウェアサポート、コントローラー解決、セッション認証、htmlレンダリングを完備しhttpの`Route`アーキテクチャを反映した専用のイベントルーターを提供します。ソケットサーバーはhttpサーバーと並行して自動的に起動するため個別にインストールするものは何もありません。

クライアントサイドではフレームワークのフロントエンドランタイムが接続、イベント送信、イベント待機を管理するグローバルな`Socket`ファサードを公開します。ソケット接続はspaルーターの初期化時に自動的に確立されます。

このソケットレイヤー上に構築される`d-wire`および`d-live`リアクティブ要素の詳細については[リアクティビティ](../frontend/reactivity.md)のドキュメントを参照してください。

<a name="仕組み"></a>

## 仕組み

アプリケーションが起動するとフレームワークは`/ws`パスでhttpサーバーにソケットアップグレードハンドラーをアタッチします。ブラウザがwebsocket接続を開くとフレームワークはアップグレードリクエストから完全な`Request`オブジェクトを生成しソケットイベントハンドラー内でクッキー、セッション、ヘッダー、認証済みユーザーへのアクセスを提供します。

受信メッセージは`event`プロパティを持つjsonオブジェクトとして期待されます。ソケットルーターはそのイベント名に登録されたハンドラーを検索しhttpルートが処理されるのとまったく同様にミドルウェアパイプラインを通じて実行します。

<a name="ソケットイベントの定義"></a>

## ソケットイベントの定義

ソケットイベントはルートファイル（通常は`routes/wire.js`）で定義します。基盤となる`SocketRouter`インスタンスに処理を委譲するグローバルな`Socket`ファサードを使用してイベントを登録します。

<a name="イベントハンドラー"></a>

### イベントハンドラー

`Socket.on()`メソッドは名前付きイベントのハンドラーを登録します。イベント名は慣例としてコロン区切りの名前空間を使用します。

```javascript
Socket.on('chat:message', (req) => {
  const { text } = req.body;
  return json({ received: true, text });
});
```

ハンドラーはhttpルートハンドラーと同じ引数を受け取ります。`req`には`req.body`（イベントペイロード）、`req.user`、`req.session`、`req.cookies`、およびすべての標準リクエストメソッドへのアクセスを含む完全なリクエストオブジェクトが含まれます。グローバルレスポンスヘルパー（`render()`、`json()`、`redirect()`、`abort()`、`cookie()`）はhttpコントローラーと同様にソケットハンドラー内でも機能します。

ハンドラーがいずれのレスポンスメソッドも呼び出さずに値を返した場合フレームワークは自動的にステータスコード200のjsonレスポンスでラップします。

<a name="インラインハンドラー"></a>

### インラインハンドラー

単純なイベントの場合はインライン関数を直接使用できます。

```javascript
Socket.on('ping', () => {
  return json({ pong: true, time: Date.now() });
});
```

<a name="コントローラーハンドラー"></a>

### コントローラーハンドラー

複雑なロジックの場合は使い慣れた`Controller@method`文字列表記を使用してコントローラーメソッドを参照します。フレームワークはコントローラーを動的に解決してインポートします。

```javascript
Socket.on('chat:message', 'ChatController@onMessage');
Socket.on('chat:typing', 'ChatController@onTyping');
```

```javascript
export default class ChatController {
  async onMessage(req) {
    const { text } = req.body;
    await Message.create({ user_id: req.user.id, text });
    Socket.broadcast('chat:new', { text, user: req.user.name });
  }

  onTyping() {
    return json({ typing: true });
  }
}
```

ソケットwireコントローラーはhttpコントローラーと同じ`controllers`ディレクトリに保存され同じ規約に従います。単一のコントローラーでhttpルートとソケットイベントの両方を処理できます。

<a name="ミドルウェア"></a>

## ミドルウェア

ソケットイベントはhttpルートと同じミドルウェアシステムをサポートします。ミドルウェア関数は`req`と`next`を受け取りハンドラーに進むために`next()`を呼び出す必要があります。

<a name="インラインミドルウェア"></a>

### インラインミドルウェア

最後の要素がハンドラーでそれ以前のすべての要素がミドルウェア参照である配列を渡すことで単一のイベントにミドルウェアをアタッチできます。

```javascript
Socket.on('admin:action', ['AuthMiddleware@requireAuth', 'AdminController@execute']);
```

<a name="グループ化ミドルウェア"></a>

### グループ化ミドルウェア

`Socket.group()`メソッドは共有ミドルウェアを複数のイベントに一度に適用します。

```javascript
Socket.group({ middleware: 'AuthMiddleware@requireAuth' }, (auth) => {
  auth.on('profile:update', 'ProfileController@update');
  auth.on('profile:avatar', 'ProfileController@avatar');
});
```

より流暢なapiのために`Socket.middleware()`メソッドを使用してミドルウェアをチェーンすることもできます。

```javascript
Socket.middleware('AuthMiddleware@requireAuth').group({}, (auth) => {
  auth.on('dashboard:refresh', 'DashboardController@refresh');
});
```

実際のアプリケーションにおけるソケットイベントの例は以下のとおりです。

```javascript
import { Socket } from 'dframework';

Socket.on('pair:status', 'auth.AuthController@wire_status').targets(['#qr-box']);

Socket.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.on('follow:toggle', 'app.FriendsController@toggle').targets(['#leaderboard-list', '#friends-list-container']);
  auth.on('friends:tab', 'app.FriendsController@friends_tab').targets(['#friends-list-container']);
  auth.on('playback:heartbeat', 'app.PlaybackController@handleHeartbeat');
});
```

<a name="イベントへの応答"></a>

## イベントへの応答

ソケットハンドラーはhttpコントローラーで利用可能なものと同じグローバルレスポンスヘルパーを使用します。すべてのレスポンスはソケット接続経由で配信されます。

<a name="HTMLのレンダリング"></a>

### HTMLのレンダリング

`render()`ヘルパーはビューテンプレートをコンパイルし結果のhtmlをクライアントに送信します。フレームワークはページ全体をリフレッシュすることなくクライアントサイドの指定されたターゲット要素にhtmlをモーフィングします。

```javascript
Socket.on('feed:refresh', async () => {
  const posts = await Post.latest(20);
  return render('partials.feed', { posts });
});
```

レンダリングされたhtmlは`d-wire:render`イベントとして送信されます。クライアントサイドのランタイムがそれを解析しフォーカス状態、入力値、スクロール位置を保持したままターゲットdom要素をモーフィングします。

フレームワークは不要なdom更新を防ぐためにペイロードハッシュを使用します。レンダリングされたhtmlがそのターゲットの前回レンダリングと同じハッシュを生成する場合更新は静かにスキップされます。

<a name="JSONの送信"></a>

### JSONの送信

`json()`ヘルパーはjsonペイロードをクライアントに返送します。

```javascript
Socket.on('stats:fetch', async () => {
  const count = await User.count();
  return json({ users: count });
});
```

<a name="リダイレクト"></a>

### リダイレクト

`redirect()`ヘルパーはリダイレクト指示をクライアントに送信します。ブラウザは指定されたurlに遷移します。

```javascript
Socket.on('session:expired', () => {
  return redirect('/login');
});
```

<a name="処理の中断"></a>

### 処理の中断

`abort()`ヘルパーはステータスコードとオプションのメッセージを含むエラーレスポンスを送信します。

```javascript
Socket.on('admin:action', (req) => {
  if (!req.user?.isAdmin) {
    return abort(403, 'forbidden');
  }
});
```

デフォルトのステータスメッセージは`400 Bad Request`、`401 Unauthorized`、`403 Forbidden`、`404 Not Found`、`429 Too Many Requests`、`500 Internal Server Error`です。

<a name="クッキー"></a>

### クッキー

`cookie()`ヘルパーを使用してソケットハンドラーからクッキーを設定できます。クッキーはレスポンスとともに送信されクライアントサイドのランタイムがブラウザに適用します。

```javascript
Socket.on('theme:set', (req) => {
  cookie('theme', req.body.theme, { path: '/', sameSite: 'Lax' });
  return json({ ok: true });
});
```

<a name="ターゲット制限"></a>

## ターゲット制限

`targets()`チェーンメソッドを使用してイベントがレンダリングを許可されるdomターゲットセレクターを制限できます。クライアントが許可リストに含まれていないターゲットを送信した場合イベントは静かに拒否されます。

```javascript
Socket.on('sidebar:update', 'SidebarController@render').targets('#sidebar');
Socket.on('panel:refresh', 'PanelController@render').targets(['#left-panel', '#right-panel']);
```

<a name="ブロードキャスト"></a>

## ブロードキャスト

`Socket.broadcast()`メソッドは接続されているすべてのwebsocketクライアントにイベントを送信します。オプションで特定の接続（通常は送信者）を除外できます。

```javascript
Socket.on('chat:send', (req, res, ws) => {
  const message = { text: req.body.text, user: req.user.name };
  Socket.broadcast('chat:new', message, ws);
  return json({ sent: true });
});
```

第3引数はブロードキャストから除外するwebsocket接続です。現在の`ws`を渡すことで送信者が自身のメッセージを受信するのを防ぐことができます。

`jobs/`内のバックグラウンドジョブから`Socket.broadcast()`および`Socket.setState()`を直接呼び出すこともできます。ワーカースレッドはredisのような外部ブローカーを必要とすることなくソケットイベントをメインプロセスに自動的にリレーします。

```javascript
// inside any background job
Socket.broadcast('task:complete', { taskId: 42 });
Socket.setState('#task-status', { done: true });
```

<a name="ターゲット配信"></a>

## ターゲット配信

全員にブロードキャストするのではなく特定のクライアントにメッセージを送信する必要がある場合dframeworkは`Socket`ファサード上で対象指定メソッドを提供します。接続は接続時および認証済みソケットリクエスト中に認証済みユーザーのIDとセッションIDを自動的に追跡します。

<a name="特定ユーザーへの送信"></a>

### 特定ユーザーへの送信

`Socket.toUser()`メソッドはユーザーIDまたはユーザーモデルインスタンスに属するすべてのオープンなwebsocket接続にイベントを送信します。

```javascript
// using numeric user id
Socket.toUser(42, 'notification:new', { text: 'you received a payment' });

// using model instance
Socket.toUser(user, 'notification:new', { text: 'profile updated' });

// push component state directly
Socket.toUserState(42, '#cart-count', { count: 3 });

// fluent chaining syntax
Socket.toUser(42).emit('notification:new', { text: 'order shipped' });
Socket.toUser(42).setState('#cart-count', { count: 3 });
```

<a name="複数ユーザーへの送信"></a>

### 複数ユーザーへの送信

`Socket.toUsers()`メソッドはユーザーIDまたはユーザーモデルインスタンスのリストにイベントをディスパッチします。

```javascript
// dispatch event to multiple users
Socket.toUsers([1, 2, 3], 'room:update', { active: true });

// push component state directly to multiple users
Socket.toUsersState([1, 2, 3], '#status', { online: true });

// fluent chaining syntax
Socket.toUsers([1, 2, 3]).emit('room:update', { active: true });
Socket.toUsers([1, 2, 3]).setState('#status', { online: true });
```

<a name="セッションへの送信"></a>

### セッションへの送信

`Socket.toSession()`メソッドは特定のセッションIDに一致するすべての接続を対象とします。

```javascript
// dispatch event to specific session
Socket.toSession(sessionId, 'session:revoked', { reason: 'logged in from new device' });

// push component state directly to specific session
Socket.toSessionState(sessionId, '#auth-panel', { authenticated: false });

// fluent chaining syntax
Socket.toSession(sessionId).emit('session:revoked', { reason: 'logged in from new device' });
Socket.toSession(sessionId).setState('#auth-panel', { authenticated: false });
```

<a name="ダイレクトソケットターゲット"></a>

### ダイレクトソケットターゲット

`Socket.to()`ヘルパーはwebsocketインスタンスを直接対象とします。

```javascript
Socket.to(ws).emit('direct:message', { ok: true });
Socket.to(ws, 'direct:message', { ok: true });
Socket.to(ws).setState('#chat-box', { focused: true });
```

<a name="マルチガードターゲット"></a>

### マルチガードターゲット

アプリケーションが`config/auth.js`で複数の認証ガード（例えば`web`、`admin`、`partner`）を定義している場合接続は設定されたすべてのガードにわたってIDを自動的に追跡します。

`Socket.toUser(id)`は任意のガード（またはデフォルトガード）上でそのユーザーIDに一致する接続に配信します。特定のガードを明示的に対象とするには`Socket.guard()`を使用してディスパッチのスコープを制限するか直接ガードメソッドを呼び出すことができます。

```javascript
// target single user or multiple users through the guard scope
Socket.guard('admin').toUser(5, 'admin:alert', { text: 'server restart scheduled' });
Socket.guard('admin').toUsers([5, 6], 'admin:alert', { text: 'maintenance incoming' });

// fluent state updates scoped by guard
Socket.guard('admin').toUser(5).setState('#admin-panel', { status: 'maintenance' });
Socket.guard('admin').toUsers([5, 6]).setState('#admin-panel', { status: 'maintenance' });

// direct guard dispatch methods
Socket.toGuardUser('admin', 5, 'admin:alert', { text: 'server restart scheduled' });
Socket.toGuardUsers('admin', [5, 6], 'admin:alert', { text: 'server restart scheduled' });

// direct guard component state updates
Socket.toGuardUserState('admin', 5, '#admin-panel', { status: 'maintenance' });
Socket.toGuardUsersState('admin', [5, 6], '#admin-panel', { status: 'maintenance' });

// fluent chaining on direct guard methods
Socket.toGuardUser('admin', 5).emit('admin:alert', { text: 'server restart scheduled' });
Socket.toGuardUser('admin', 5).setState('#admin-panel', { status: 'maintenance' });
Socket.toGuardUsers('admin', [5, 6]).emit('admin:alert', { text: 'server restart scheduled' });
Socket.toGuardUsers('admin', [5, 6]).setState('#admin-panel', { status: 'maintenance' });
```

<a name="ユーザー接続の取り消し"></a>

### ユーザー接続の取り消し

`Socket.revokeUser()`メソッドはすべてのガードまたは特定のガードにわたってユーザーのアクティブなソケット接続を即座に認証解除します。

```javascript
// revoke user on all guards
Socket.revokeUser(userId);

// revoke user only on a specific guard
Socket.revokeUser(adminId, 'admin');
```

すべての対象指定メソッド（`toUser`、`toUserState`、`toUsers`、`toUsersState`、`toSession`、`toSessionState`、`to`、`guard`、`toGuardUser`、`toGuardUserState`、`toGuardUsers`、`toGuardUsersState`、`revokeUser`）はhttpコントローラー、wireハンドラー、バックグラウンドジョブ全体でシームレスに動作します。

<a name="Socketファサード"></a>

## Socketファサード

グローバルの`Socket`はすべてのメソッド呼び出しをアプリケーションの`SocketRouter`インスタンスに委譲するプロキシファサードです。アプリケーションの初期化後にグローバルに利用可能でありフレームワークからインポートすることもできます。

```javascript
import { Socket } from 'dframework';
```

ファサードはすべてのメソッドを公開します: `on()`、`group()`、`middleware()`、`broadcast()`、`setState()`、`to()`、`toUser()`、`toUserState()`、`toUsers()`、`toUsersState()`、`toSession()`、`toSessionState()`、`guard()`、`toGuardUser()`、`toGuardUserState()`、`toGuardUsers()`、`toGuardUsersState()`、`revokeUser()`、`liveNotify()`。

<a name="クライアントサイドでの使用"></a>

## クライアントサイドでの使用

フレームワークのフロントエンドランタイムはwebsocket接続を管理するグローバルな`Socket`オブジェクトを`window`上に提供します。

<a name="接続"></a>

### 接続

websocket接続はspaルーターの初期化時に自動的に確立されます。手動で接続することもできます。

```javascript
await Socket.connect();
```

接続urlはデフォルトで`/ws`でありセキュアなページでは自動的に`wss:`をセキュアでないページでは`ws:`を使用します。完全なuriは現在のページの`location.host`から構築されます。

`Socket`フロントエンドファサードはスコープ付きスクリプト（`type="text/dspa"`、インラインページスクリプト、および`d-spa-scope`付きモジュールスクリプト）内でのみ利用可能です。スコープコンテキスト外では利用できません。

<a name="イベントの送信"></a>

### イベントの送信

サーバーにイベントを送信するには`Socket.emit()`を使用します。データペイロードとオプションのターゲットセレクターを渡すことができます。

```javascript
Socket.emit('chat:send', { text: 'hello world' });
```

```javascript
Socket.emit('dashboard:refresh', { filter: 'today' }, '#dashboard');
```

ターゲットが指定されている場合サーバーハンドラーのレンダリング出力はその特定のdom要素にモーフィングされます。

<a name="イベントの待機"></a>

### イベントの待機

サーバーからプッシュされたイベントを受信するにはクライアントサイドで`Socket.on()`を使用します。

```javascript
Socket.on('chat:new', (data) => {
  appendMessage(data.text, data.user);
});
```

ブロードキャストイベントの場合コールバックはデータペイロードを直接受け取ります。

<a name="オリジン保護"></a>

## オリジン保護

> [!IMPORTANT]
> ソケットサーバーはクロスサイトwebsocketハイジャック（cswsh）攻撃を防ぐためすべてのアップグレードリクエストの`Origin`ヘッダーを`APP_URL`に対して検証します。不一致のオリジンからの接続は即座に拒否されます。

`local`環境では設定されたurlに関係なく`localhost`および`127.0.0.1`からの接続が許可されます。

<a name="デバッグモード"></a>

## デバッグモード

デバッグモードが有効な`local`環境ではソケットルーターはすべてのレスポンスに詳細なタイミング情報とクエリ情報を付加します。クライアント側のdebugbarには各ソケットイベントの経過時間、実行されたクエリ、リクエストヘッダーが表示されhttpリクエストと同等の可視性が提供されます。
