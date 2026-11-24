# テスト

- [テスト](#テスト)
  - [はじめに](#はじめに)
  - [テストの実行](#テストの実行)
  - [テストの生成](#テストの生成)
  - [http統合テスト](#http統合テスト)
    - [リクエストの送信](#リクエストの送信)
    - [名前付きルートのディスパッチ](#名前付きルート)
    - [コントローラーの直接実行](#コントローラーの直接実行)
    - [ユーザーとしての認証](#ユーザーとしての認証)
    - [利用可能なアサーション](#利用可能なアサーション)
  - [サービスフェイク](#サービスフェイク)
    - [バックグラウンドジョブのフェイク](#バックグラウンドジョブのフェイク)
    - [websocketイベントのフェイク](#websocketイベントのフェイク)
  - [データベースアサーション](#データベースアサーション)
    - [マイグレーション、シーダー、およびトランザクション](#マイグレーションとトランザクション)
  - [ブラウザとページ操作テスト](#ブラウザとページ操作テスト)
    - [ページのナビゲーション](#ページのナビゲーション)
    - [フォーム操作とクリック](#フォーム操作とクリック)
    - [ブラウザアサーション](#ブラウザアサーション)

<a name="はじめに"></a>

## はじめに

dframeworkには外部依存ゼロの組み込みテストハーネスが含まれており、nodeのネイティブテストランナー（`node:test`）上で直接動作します。ネットワークソケットを起動することなく高速なインプロセスhttpリクエストを実行し、データベースの状態を検証し、ci環境で軽量なシミュレーテッドdomを用いてフォーム送信や画面遷移を完全にテストできます。

`dframework/testing`モジュールはトップレベルのテストランナーおよびアサーションプリミティブ（`test`、`it`、`describe`、`Tester`、厳格な`assert`）をエクスポートします。テストファイルはボイラープレート不要の統一された`Tester`ファサード（`Tester.get`、`Tester.post`、`Tester.route`、`Tester.actingAs`、`Tester.fake`、`Tester.assertDatabaseHas`）を使用します。他のファサードと同様に、`Tester`はテスト環境のglobalThisでもグローバルに利用可能です。テストランナーは各種修飾子をサポートしています: フォーカス実行用の`test.only()`、一時的にスキップする`test.skip()`、未実装の仕様を宣言する`test.todo()`（コールバック関数の有無にかかわらず指定可能）、データベースの変更を自動ロールバックする`test.transaction()`。

<a name="テストの実行"></a>

## テストの実行

`dstrn test`を使用してテストスイート全体または特定のファイルを対象にテストを実行できます:

```bash
# tests/内のすべてのテストを実行
dstrn test

# 特定のテストファイルまたはディレクトリを実行
dstrn test tests/feature/UserTest.test.js
dstrn test tests/unit

# ファイル変更時に自動再実行するウォッチモード
dstrn test --watch

# { only: true }が付与されたテストのみを実行
dstrn test --only
```

`dstrn test`は自動的に`APP_ENV=testing`を設定し、node 22または23で実行されている場合は実験的なsqliteサポートを有効にします。テストの実行にはプロジェクトルートに`.env.testing`ファイルが存在することが必須です。`.env`へのフォールバックは行われないため、開発や本番のデータベースに対する偶発的なデータ変更を防止します。

<a name="テストの生成"></a>

## テストの生成

`dstrn make:test`を使用して新しいテストファイルをスキャフォールディングできます:

```bash
# tests/feature/に機能テストを作成
dstrn make:test UserTest
dstrn make:test UserTest --feature

# tests/browser/にブラウザテストを作成
dstrn make:test CheckoutTest --browser

# tests/unit/に単体テストを作成
dstrn make:test CalculatorTest --unit

# ネストされたディレクトリも自動作成
dstrn make:test Auth/RegistrationTest
```

<a name="http統合テスト"></a>

## http統合テスト

`Tester`ファサードは、フレームワークのパイプラインに合成リクエストストリームを直接渡すインプロセスhttpディスパッチャーを提供します。これによりモックの乖離を防ぎながらネットワークソケットのリッスンオーバーヘッドを排除します。

<a name="リクエストの送信"></a>

### リクエストの送信

```javascript
import { test, Tester } from 'dframework/testing';

test('アクティブなユーザーを返す', async () => {
  const res = await Tester.get('/api/users');

  res.assertOk();
  res.assertJson({ success: true });
});
```

サポートされているリクエストメソッドには`get`、`post`、`put`、`patch`、`delete`、`head`、`options`、および`Tester.request(method, uri, data, headers)`によるカスタムメソッドがあります。同一クライアントインスタンス内での連続したリクエスト間ではcookieとセッション状態が維持されます。

カスタムヘッダーやクッキーを設定したり、csrfトークンの自動生成を無効化することもできます:

```javascript
// カスタムヘッダーやクッキーを設定
Tester.withHeaders({ 'x-test-header': 'custom' });
Tester.withCookies({ theme: 'dark' });

// 更新系リクエストでのcsrfトークン自動生成を無効化
Tester.withoutCsrf();
```

<a name="名前付きルート"></a>

### 名前付きルートのディスパッチ

定義されたルート名とパラメータマップを使用してリクエストをディスパッチできます:

```javascript
test('名前付きルートでの注文参照', async () => {
  const res = await Tester.route('orders.show', { id: 42 });

  res.assertOk();
  res.assertJson({ orderId: '42' });
});
```

<a name="コントローラーの直接実行"></a>

### コントローラーの直接実行

`Tester.call()`を使用してコントローラーのアクションやミドルウェアチェーンを直接テストできます:

```javascript
test('コントローラーアクションの直接実行', async () => {
  const res = await Tester.call('UserController@show', {
    uri: '/users/:id',
    params: { id: 1 }
  });

  res.assertOk();
});
```

<a name="ユーザーとしての認証"></a>

### ユーザーとしての認証

リクエストを実行する前に`actingAs`を使用してユーザーを認証できます:

```javascript
test('認証済みプロフィールの表示', async () => {
  Tester.actingAs({ id: 1, email: 'alice@example.com' });

  const res = await Tester.get('/profile');
  res.assertOk();
  res.assertSee('alice@example.com');
});
```

<a name="利用可能なアサーション"></a>

### 利用可能なアサーション

合成リクエストによって返されるレスポンスオブジェクトはチェーン可能なアサーションヘルパーを提供します:

| アサーション              | 説明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `assertStatus(code)`      | レスポンスステータスコードを検証                             |
| `assertOk()`              | ステータス200を検証                                          |
| `assertCreated()`         | ステータス201を検証                                          |
| `assertNotFound()`        | ステータス404を検証                                          |
| `assertForbidden()`       | ステータス403を検証                                          |
| `assertUnauthorized()`    | ステータス401を検証                                          |
| `assertRedirect(uri)`     | リダイレクトステータスと遷移先uriを検証                      |
| `assertJson(expected)`    | レスポンスjsonに期待される構造または値が含まれているかを検証 |
| `assertJsonMissing(key)`  | レスポンスjsonから指定キーが存在しないことを検証             |
| `assertSee(text)`         | レスポンス本文に指定文字列が含まれているかを検証             |
| `assertDontSee(text)`     | レスポンス本文に指定文字列が含まれていないことを検証         |
| `assertHeader(name, val)` | レスポンスヘッダーの値を検証                                 |

<a name="サービスフェイク"></a>

## サービスフェイク

dframeworkは`Tester.fake({ jobs: true, socket: true })`（または`Tester.fakeJobs()` / `Tester.fakeSocket()`、あるいはサービス名を直接指定する`Tester.fake('jobs')`や`Tester.fake(['jobs', 'socket'])`）を通じてバックグラウンドジョブとwebsocketイベントのテストダブルおよびスパイを提供します。フェイクはメモリ上でディスパッチを捕捉し、実際の実行を防ぎ、表現力豊かなアサーションを提供し、テスト終了時に状態を自動復元します。

<a name="バックグラウンドジョブのフェイク"></a>

### バックグラウンドジョブのフェイク

`Tester.fake({ jobs: true })`（または`Tester.fakeJobs()`）を呼び出すと`Job.dispatch()`への呼び出しがインターセプトされ、検証用にディスパッチされたジョブが記録されます:

```javascript
import { test, Tester } from 'dframework/testing';

test('チェックアウト時に確認メールをディスパッチする', async () => {
  Tester.fake({ jobs: true });

  const res = await Tester.post('/checkout', { cartId: 10 });
  res.assertOk();

  Tester.assertDispatched('SendOrderConfirmationJob');
  Tester.assertDispatched('SendOrderConfirmationJob', (payload) => payload.cartId === 10);
  Tester.assertDispatchedTimes('SendOrderConfirmationJob', 1);
  Tester.assertNotDispatched('CancelOrderJob');
});
```

| アサーション | 説明 |
| --- | --- |
| `Tester.assertDispatched(job, callback)` | ジョブがディスパッチされたことを検証（コールバックでペイロード検証可能） |
| `Tester.assertNotDispatched(job, callback)` | ジョブがディスパッチされなかったことを検証 |
| `Tester.assertDispatchedTimes(job, count)` | ジョブが指定回数ディスパッチされたことを検証 |
| `Tester.assertNothingDispatched()` | バックグラウンドジョブがディスパッチされていないことを検証 |
| `Tester.dispatched(job, callback)` | フィルターに一致するディスパッチ済みジョブを取得 |

<a name="websocketイベントのフェイク"></a>

### websocketイベントのフェイク

`Tester.fake({ socket: true })`（または`Tester.fakeSocket()`）を呼び出すと、実行中のwebsocketサーバーを必要とせずにリアルタイムのwebsocketブロードキャストと特定ユーザー宛てのメッセージをインターセプトできます:

```javascript
import { test, Tester } from 'dframework/testing';

test('メッセージをブロードキャストし受信者に通知する', async () => {
  Tester.fake({ socket: true });

  await Tester.post('/messages', { toUser: 42, text: 'hello' });

  Tester.assertBroadcast('message.sent');
  Tester.assertSentToUser(42, 'notification.new');
  Tester.assertNotSentToUser(99, 'notification.new');
});
```

| アサーション | 説明 |
| --- | --- |
| `Tester.assertBroadcast(event, callback)` | 全クライアントにブロードキャストされたことを検証 |
| `Tester.assertNotBroadcast(event, callback)` | ブロードキャストされなかったことを検証 |
| `Tester.assertSentToUser(userOrId, event, callback)` | 特定ユーザーidに送信されたことを検証 |
| `Tester.assertNotSentToUser(userOrId, event, callback)` | ユーザーに送信されなかったことを検証 |
| `Tester.assertSentToSession(sessionId, event, callback)` | 特定セッションに送信されたことを検証 |
| `Tester.assertSent(event, callback)` | いずれかのチャネルで送信されたことを検証 |
| `Tester.assertNotSent(event, callback)` | 送信されなかったことを検証 |
| `Tester.assertSentTimes(event, count)` | 指定回数送信されたことを検証 |
| `Tester.assertNothingSent()` | socketイベントが送信されていないことを検証 |
| `Tester.assertRevoked(userOrId, guardName)` | ユーザーのwebsocketセッションが無効化されたことを検証 |
| `Tester.sent(event, callback)` | フィルターに一致する送信済みsocketイベントを取得 |
| `Tester.broadcasted(event, callback)` | フィルターに一致するブロードキャスト済みsocketイベントを取得 |

<a name="データベースアサーション"></a>

## データベースアサーション

`Tester`ファサードは非同期のデータベースアサーションヘルパーを公開しています:

```javascript
import { test, Tester } from 'dframework/testing';

Tester.useDatabaseTransactions();

test('ユーザー登録がデータベースに記録される', async () => {
  await Tester.post('/register', {
    name: 'bob',
    email: 'bob@example.com'
  });

  await Tester.assertDatabaseHas('users', { email: 'bob@example.com' });
  await Tester.assertDatabaseMissing('users', { email: 'charlie@example.com' });
  await Tester.assertDatabaseCount('users', 1);
});
```

<a name="マイグレーションとトランザクション"></a>

### マイグレーション、シーダー、およびトランザクション

専用のヘルパーでデータベース状態を管理し、トランザクションを使用してテストによる変更を自動的に分離できます:

```javascript
test('データベーストランザクションとシーディング', async () => {
  await Tester.migrate();
  await Tester.rollback();
  await Tester.migrate();
  await Tester.seed('UserSeeder');

  await Tester.useTransaction(async () => {
    await Tester.post('/users', { name: 'Alice' });
    await Tester.assertDatabaseHas('users', { name: 'Alice' });
  });

  await Tester.assertDatabaseMissing('users', { name: 'Alice' });
});
```

<a name="ブラウザとページ操作テスト"></a>

## ブラウザとページ操作テスト

`browse()`クライアントは軽量なシミュレーテッドdomパーサーを実行し、ci環境で重いヘッドレスブラウザバイナリを必要とせずにフォームのシリアライズ、リンクの遷移、入力のタイピング、チェックボックス、セレクトボックスを忠実にシミュレートします。なおシミュレーテッドdomはサーバーレンダリングされたhtmlを空白区切りの子孫セレクターおよび`:not()`で解釈する軽量パーサーであり、クライアント側のjavascriptランタイムや複合結合子（`>`、`+`、`~`など）は実行されません。

<a name="ページのナビゲーション"></a>

### ページのナビゲーション

```javascript
import { test, browse } from 'dframework/testing';

test('訪問してリンクをクリックし遷移する', async () => {
  const page = await browse('/');

  page.assertSee('welcome to dframework');
  page.assertElement('nav');

  await page.click('#docs-link');
  page.assertPath('/docs');
});
```

<a name="フォーム操作とクリック"></a>

### フォーム操作とクリック

```javascript
test('お問い合わせフォームの入力と送信', async () => {
  const page = await browse('/contact');

  page.type('input[name="email"]', 'visitor@example.com');
  page.type('textarea[name="message"]', 'hello framework');
  page.check('input[name="newsletter"]');
  page.select('select[name="topic"]', 'support');

  await page.click('button[type="submit"]');

  page.assertPath('/contact/success');
  page.assertSee('thank you for reaching out');
});
```

<a name="ブラウザアサーション"></a>

### ブラウザアサーション

| アサーション                   | 説明                                                             |
| ------------------------------ | ---------------------------------------------------------------- |
| `assertPath(path)`             | 現在のurlパス名を検証                                            |
| `assertSee(text)`              | レンダリングされたテキストに指定文字列が含まれているかを検証     |
| `assertDontSee(text)`          | レンダリングされたテキストに指定文字列が含まれていないことを検証 |
| `assertElement(selector)`      | cssセレクタに一致する要素が存在することを検証                    |
| `assertMissing(selector)`      | cssセレクタに一致する要素が存在しないことを検証                  |
| `assertValue(selector, value)` | inputまたはselect要素が期待される値を持つことを検証              |
| `assertChecked(selector)`      | チェックボックスがチェックされていることを検証                   |
| `assertNotChecked(selector)`   | チェックボックスがチェックされていないことを検証                 |
