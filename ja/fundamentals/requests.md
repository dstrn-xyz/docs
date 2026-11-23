# リクエスト

- [はじめに](#はじめに)
- [リクエストコンテキスト](#リクエストコンテキスト)
- [リクエストの識別](#リクエストの識別)
- [入力とパラメータ](#入力とパラメータ)
- [ストリーミングリクエスト](#ストリーミングリクエスト)
- [ヘッダー](#ヘッダー)
- [csrf管理](#csrf管理)
- [リクエストトラッキング](#リクエストトラッキング)

<a name="はじめに"></a>

## はじめに

`Request`クラスは生のnode httpリクエストをラップし、受信データを操作するためのapiを提供します。url、クエリ文字列、ヘッダー、ボディペイロード（json、form urlencoded、マルチパートファイル）を解析します。

コントローラーおよびミドルウェアでは、リクエストインスタンスが`req`引数として注入されます。

<a name="リクエストコンテキスト"></a>

## リクエストコンテキスト

フレームワークは非同期呼び出しスタック全体でリクエスト状態を維持するために`AsyncLocalStorage`を利用します。これにより、すべての関数に`req`オブジェクトを引き渡すことなく、現在のリクエストへグローバルにアクセスできます。

```javascript
import { RequestContext } from 'dframework';

function getClientIp() {
  const req = RequestContext.get();
  return req ? req.headers['x-forwarded-for'] || req.socket.remoteAddress : null;
}
```

<a name="リクエストの識別"></a>

## リクエストの識別

リクエストオブジェクトは、リクエストの送信元とフォーマットを判定するための複数のゲッターを提供します。

```javascript
// returns true if the request comes from native builds
if (req.isNative) {
  return json({ success: true, native: true });
}

// returns true for XMLHttpRequest, dSPAHttpRequest, or application/json accept headers
if (req.isAjax) {
  return abort(422, 'validation failed');
}

// returns client ip address (honors X-Forwarded-For / X-Real-IP when app.trustProxy is enabled)
const clientIp = req.ip;

// returns true specifically for the framework's client side spa router
if (req.isSPA) {
  return json({ component: 'Home' });
}
```

<a name="入力とパラメータ"></a>

## 入力とパラメータ

フレームワークは入力を必要としないルートでのオーバーヘッドを回避するため、入力を遅延パースします。

```javascript
// parsed from the query string (e.g. ?search=term)
const query = req.query.search;

// parsed from the cookie header
const locale = req.cookies.locale;

// automatically extracts and ensures 'page' is an integer, defaulting to 1
const page = req.page;

// retrieve unified input across params, body, and query (priority: params > body > query)
const name = req.input('name', 'guest');
const role = req.input('role');

// retrieve all merged inputs
const allInputs = req.all(); // or req.input()

// retrieve uploaded file instance (first item if array)
const avatar = req.file('avatar');

// check if a file was uploaded for the given key
if (req.hasFile('document')) {
  const doc = req.file('document');
}
```

[コントローラーのドキュメント](controllers.md)で詳述されているように、変更系リクエスト（`POST`、`PUT`、`PATCH`、`DELETE`）ではルート上でcsrf、shield、またはボディへのアクセスが有効な場合に`req.body`と`req.files`が自動的に設定されます。csrfとshieldの両方を無効にしたルートで静的解析に検出されない間接的なパターンからボディにアクセスする場合は、[ルーティング](routing.md#プロファイル)で説明されているように`.profile({ needsBody: true })`を設定してください。また、`req.file(key)`を使用して最初のファイルを直接取得したり、`req.hasFile(key)`を使用してアップロードされたファイルが存在するか確認したりすることもできます。

<a name="ストリーミングリクエスト"></a>

## ストリーミングリクエスト

チャンクをメモリに蓄積することなく、生の受信リクエストペイロードを直接消費またはパイプするには、読み取り可能なストリームを取得できます:

```javascript
const stream = req.createReadableStream();
if (stream) {
  for await (const chunk of stream) {
    // process chunk
  }
}
```

<a name="ヘッダー"></a>

## ヘッダー

大文字と小文字を区別しない`header()`メソッドを使用してヘッダーを取得できます。

```javascript
const userAgent = req.header('user-agent');
const referer = req.header('referrer'); // handles both referer and referrer
```

<a name="csrf管理"></a>

## csrf管理

リクエストオブジェクトはcsrfトークンの生成と検証を処理します。

> [!NOTE]
> 明示的に無効化されていない限り、ルートコンパイラはすべての変更系ルート（`POST`、`PUT`、`PATCH`、`DELETE`）に対して検証ロジックを自動的に挿入するため、これらのメソッドを手動で呼び出すことはほとんどありません。

```javascript
// generate a new token valid for 6 hours
const token = req.csrfToken();

// verify a submitted token
const isValid = req.verifyCsrfToken(submittedToken);

// rotate the csrf secret and generate a fresh token (invalidates previous tokens)
const newToken = req.rotateCsrfToken();
```

<a name="リクエストトラッキング"></a>

## リクエストトラッキング

トラッキング対象の各リクエストには`req.trackingId`でアクセス可能な関連付けidが割り当てられます。また`req.capture()`を使用すると、実行時間、ヘッダー、クエリ、ボディ、ユーザー、実行されたデータベースクエリ、ディスパッチされたバックグラウンドジョブ、エラー詳細を含むリクエスト状態の完全なスナップショットをいつでも取得できます。

```javascript
export default class CheckoutController {
  process(req) {
    const trackingId = req.trackingId;
    const snapshot = req.capture();
    return json({ trackingId, success: true });
  }
}
```
