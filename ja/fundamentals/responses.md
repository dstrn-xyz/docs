# レスポンス

- [レスポンス](#レスポンス)

  - [はじめに](#はじめに)
  - [基本的なレスポンス](#基本的なレスポンス)
  - [ヘッダーとcookie](#ヘッダーとcookie)
  - [jsonレスポンス](#jsonレスポンス)
  - [ビューのレンダリング](#ビューのレンダリング)
  - [ストリーミングレスポンス](#ストリーミングレスポンス)
  - [リダイレクト](#リダイレクト)
  - [エラーレスポンス](#エラーレスポンス)

<a name="はじめに"></a>

## はじめに

`Response`クラスは生のnode httpレスポンスをラップします。ただし、レスポンスオブジェクトを直接操作することはほとんどありません。代わりに、コントローラーはグローバルに利用可能なレスポンスヘルパーのいずれかを返します: `render()`、`json()`、`redirect()`、`back()`、`abort()`、`status()`、または`stream()`。

dframeworkは返されたヘルパーを検出し、httpレスポンスを自動的にファイナライズします。

<a name="基本的なレスポンス"></a>

## 基本的なレスポンス

`status()`ヘルパーはhttpステータスコードを設定するために使用されます。チェーン可能なレスポンスオブジェクトを返します。`.json()`または`.send()`をチェーンできます。

```javascript
export default class WebhookController {
  async handle() {
    // responds with a 202 accepted and empty body
    return status(202).send('');
  }
}
```

<a name="ヘッダーとcookie"></a>

## ヘッダーとcookie

ファイナライズする前に、ヘッダーとcookieをレスポンスにアタッチできます。これらのメソッドはチェーン可能です。

```javascript
export default class DownloadController {
  async export() {
    const csv = await generateCsv();
    return status(200)
      .set('Content-Disposition', 'attachment; filename="export.csv"')
      .type('text/csv')
      .send(csv);
  }
}
```

cookieを設定するには、`.cookie()`メソッドを使用します。`maxAge`オプションは`Max-Age`ヘッダーを秒単位で設定します（真偽値または空文字列を渡した場合は安全に無視され、標準のセッションcookieのままとなります）。`sameSite`オプションはcookieの分離を設定します（`'lax'`、`'strict'`、`'none'`、または`true`が`SameSite=Strict`に解決される真偽値）。

```javascript
export default class ThemeController {
  async update() {
    return status(200)
      .cookie('theme', 'dark', { httpOnly: false, path: '/', maxAge: 31536000, sameSite: 'lax' })
      .json({ success: true });
  }
}
```

<a name="jsonレスポンス"></a>

## jsonレスポンス

`json()`ヘルパーは自動的に`Content-Type`を`application/json`に設定し、提供されたオブジェクトをシリアライズします。rfc 8259に従いjsonはデフォルトでutf-8であるため、明示的なcharsetは不要です。

```javascript
export default class ApiController {
  async user(req) {
    const user = await User.find(req.params.id);
    return json({ success: true, user });
  }
}
```

`.withErrors()`を介してバリデーションエラーがレスポンスにバインドされている場合、フレームワークはそれらをjsonペイロードに自動挿入します。デバッグバーが有効なローカル開発モードでアプリケーションが実行されている場合、パフォーマンスメトリクスもjsonペイロードに自動挿入されます。

<a name="ビューのレンダリング"></a>

## ビューのレンダリング

`render()`ヘルパーはビューテンプレートを評価し、生成されたhtmlを送信します。レンダリングは非同期にスケジュールされるため、htmlが実際にコンパイルされる前にデータバインディングメソッドをチェーンできます。

```javascript
export default class ProfileController {
  async show() {
    const user = Auth.user();
    
    return render('app.profile.show', { user })
      .withData({ title: 'my profile' });
  }
}
```

フォーム送信が失敗した場合、`.withErrors()`および`.withInput()`を使用してバリデーションエラーと古い入力をビューにフラッシュバックできます。また、`.with(key, value)`を使用してカスタムのキーと値のペアをフラッシュすることもできます。

```javascript
export default class SettingsController {
  async update(req) {
    const validation = validate(req.body, rules);
    
    if (validation.fails()) {
      return back().withErrors(validation.errors()).withInput(req.body).with('toast', 'settings could not be saved');
    }
  }
}
```

<a name="ストリーミングレスポンス"></a>

## ストリーミングレスポンス

`stream(writer)`ヘルパーは、バックプレッシャーサポートを備えたチャンク単位でレスポンスボディを送信します。同期または非同期関数`(write, close)`を受け取ります。ここで`write(chunk)`はチャンクを送信し、トランスポートが次のデータを受け入れる準備ができたときに解決されるプロミスを返します。`close()`はレスポンスを終了します。

ストリーミングはペイロード全体をメモリにバッファリングすることなく大きなファイル（音声、動画、ドキュメント）をプロキシするのに最適です。他のレスポンスメソッドと同様に、`stream()`を呼び出す前にヘッダー（content type、range、cache control）を設定します。

dframeworkはバックプレッシャーを自動的に処理します。クライアントまたはトランスポートが追いつかない場合、`write()`はソケットが再び書き込み可能になるまで待機してから解決します。

> [!NOTE]
> レスポンスプールは`close()`が呼び出されるまで解放されず、長時間実行されるストリームが内部オブジェクトの再利用に干渉しないようにします。ライター関数が`close()`を呼び出さずに復帰した場合、フレームワークが自動的に呼び出します。

```javascript
export default class StreamController {
  async audio(req) {
    const file = await AudioFile.find(req.params.id);
    if (!file) return abort(404);

    const upstream = await fetch(file.url, {
      headers: { range: req.headers['range'] }
    });

    return status(upstream.status)
      .set('accept-ranges', 'bytes')
      .set('content-range', upstream.headers.get('content-range'))
      .type('audio/mpeg')
      .stream(async (write, close) => {
        const reader = upstream.body.getReader();
        while (true) {
          const { done, value } = await reader.read();
          if (done) break;
          await write(Buffer.from(value));
        }
        close();
      });
  }
}
```

ライターがエラーをスローした場合、レスポンスは終了しプールは即座に解放されます。

<a name="リダイレクト"></a>

## リダイレクト

`redirect(url, code = 302, force = false)`ヘルパーはリダイレクトレスポンスを作成します。

[コントローラーのドキュメント](controllers.md)で言及されているように、dframeworkのリダイレクトヘルパーはオープンリダイレクト脆弱性に対して厳密に保護されています。クロスオリジンまたはプロトコル相対urlへのリダイレクトを拒否し、代わりにエラーをスローします（ローカル開発時を除く）。

また、spa対応でもあります。リクエストがクライアントサイドのルーター（`dSPAHttpRequest`）によって行われた場合、標準のhttp 302を発行する代わりにフロントエンドルーターが実行するためのjson命令を返します。`force`真偽値を`true`に設定すると、spaルーターに強制的なハードページリロードを実行させます。

```javascript
export default class AuthController {
  async logout() {
    await Auth.logout();
    return redirect('/home');
  }
}
```

`back()`ヘルパーはユーザーを直前の場所へリダイレクトします。以下の順序で宛先を解決します:

1. セッションに記録された直前のurl（直前の成功したGETリクエストからの`req.previousUrl`）
2. 受信したHTTP `Referer`ヘッダー（オリジンがホストと一致するか検証済み）
3. 提供された`fallback` url（デフォルトは`'/'`）

```javascript
export default class PostController {
  async destroy(req) {
    await Post.where('id', req.params.id).delete();
    return back('/fallback-url');
  }
}
```

カスタムステータスコードを渡し、フルページリダイレクトを強制することもできます:

```javascript
return back('/home', 303, true);
```

<a name="エラーレスポンス"></a>

## エラーレスポンス

`abort(code, message)`ヘルパーは実行を即座に停止しエラーレスポンスを返します。コンテキスト認識型であり、リクエストがajax経由で行われた場合はjsonエラーペイロードを自動的に返します。標準的なブラウザリクエストであった場合は適切なhtmlエラーページ（404、500など）をレンダリングするグローバルエラーハンドラーにエラーを渡します。

```javascript
export default class FileController {
  async show(req) {
    const file = await File.find(req.params.id);
    
    if (!file) {
      return abort(404, 'file not found');
    }
    
    if (file.owner_id !== Auth.user().id) {
      return abort(403, 'unauthorized access');
    }
    
    return json({ file });
  }
}
```
