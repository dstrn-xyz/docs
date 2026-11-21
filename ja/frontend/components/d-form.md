# d-form

CSRFトークンの自動注入、クライアント側バリデーション、インラインフィールドエラーレンダリング、レスポンスルーティングを備えたAjaxフォームコンテナーです。

```html
<d-form action="/register" method="POST" rules='{"email": "required|email"}' class="flex-column g-1">
  <input type="email" name="email" placeholder="email address">
  <input type="password" name="password" placeholder="password">
  <button type="submit" class="btn">register</button>
</d-form>
```

<a name="属性とプロパティ（d-form）"></a>

## 属性とプロパティ（d-form）

| 属性           | プロパティ    | 型        | デフォルト値 | フォーム | 説明                                                            |
| :------------- | :------------ | :-------- | :----------- | :------- | :-------------------------------------------------------------- |
| `action`       | `action`      | `string`  | `""`         | いいえ   | 送信先URL                                                       |
| `method`       | `method`      | `string`  | `"POST"`     | いいえ   | HTTPリクエストメソッド（`GET`、`POST`、`PUT`、`DELETE`）        |
| `rules`        | `rules`       | `string`  | `""`         | いいえ   | クライアント側バリデーションルールのJSON文字列                  |
| `target`       | `target`      | `string`  | `""`         | いいえ   | サーバーがHTMLを返した際に差し替える要素のCSSセレクター         |
| `navigate`     | `navigate`    | `boolean` | `false`      | いいえ   | ブラウザー履歴を更新しHTML返却時にSPAルーターを使用             |
| `force-reload` | `forceReload` | `string`  | `""`         | いいえ   | リロード条件（`""`、`"true"`、`"all"`、`"success"`、`"error"`） |
| `callback`     | `callback`    | `string`  | `""`         | いいえ   | レスポンス成功後に実行されるグローバルwindow関数の名前          |

<a name="プログラム用api（d-form）"></a>

## プログラム用api（d-form）

```javascript
const form = select('d-form');

form.submit();

const formData = form.serialize();

form.showErrors({ email: ['invalid address'] });
form.clearErrors();
form.setDisabled(true);
```

<a name="イベント（d-form）"></a>

## イベント（d-form）

| イベント     | 詳細                   | バブリング | 説明                                                   |
| :----------- | :--------------------- | :--------- | :----------------------------------------------------- |
| `d-submit`   | なし                   | はい       | キャンセル可能、送信開始前に送出                       |
| `d-success`  | `object`または`string` | はい       | リクエスト成功時に送出                                 |
| `d-error`    | `object`               | はい       | リクエスト失敗時またはバリデーションエラー発生時に送出 |
| `d-response` | `object`または`string` | はい       | すべての完了したレスポンスに対して送出                 |

<a name="動作仕様（d-form）"></a>

## 動作仕様（d-form）

- 内部form要素をラップまたは注入しsubmitイベントをインターセプトします。
- `X-Requested-With: dSPAHttpRequest`および`X-CSRF-TOKEN`ヘッダーを自動注入します。
- リクエストの重複を防ぐため送信処理中はすべての入力、ボタン、フォームコントロールを無効化します。
- クライアント側ルールが定義されている場合それを評価します（バリデーション失敗時は送信を中止しインラインフィールドエラーを注入します）。
- ファイルアップロードの場合は`multipart/form-data`を送信します（標準フィールドの場合は`application/x-www-form-urlencoded`を送信します）。

<a name="レスポンスライフサイクル"></a>

## レスポンスライフサイクル

| コントローラーの戻り値             | ステータス / タイプ                         | デフォルトの動作（force-reloadなし）                                                                                                             | force-reload指定時の動作                                                                                              |
| :--------------------------------- | :------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `return json({ data })`            | `200` (`application/json`)                  | ペイロードを解析し`d-success`と`d-response`を送出、`callback`が定義されている場合は実行、ページリロードなしでフォームを画面に残す                | `force-reload`、`all`、または`success`が設定されている場合はページをリロード（`error`指定時は無視）                   |
| `return json({ redirect })`        | `200` (`application/json`)                  | リロードなしで`dSPA.navigate(redirect)`を通じて遷移（`force: true`または`X-Redirect-Force`が設定されている場合は完全なページリダイレクトを実行） | SPAルーター経由で遷移（`force: true`の場合はハードリダイレクト）                                                      |
| `return json().withErrors(errors)` | `422` (`application/json`)                  | 入力欄に`.error`を付与しフィールド直下に`.error-message`（または上部に`.error-banner`）を注入、`d-error`と`d-response`を送出                     | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はインラインエラーを保持） |
| `validate()` 失敗                  | `422` (`application/json`)                  | フィールドバリデーションエラーをインラインでレンダリング、`d-error`と`d-response`を送出、フォーム状態を維持                                      | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はインラインエラーを保持） |
| `return abort(status, message)`    | `4xx`または`5xx` (`application/json`)       | フォーム上部に`.error-banner`を表示、`d-error`と`d-response`を送出、フォーム入力を再度有効化                                                     | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はバナーを表示）           |
| `return render(view, data)`        | `200` (`text/html`)                         | 返却されたマークアップを対象セレクター（デフォルトは`body`）に差し替え、スクリプトを実行、`d-success`を送出（`navigate`設定時はURLを更新）       | `force-reload`、`all`、または`success`が設定されている場合はページをリロード（`error`指定時は差し替えを実行）         |
| 予期しないサーバーエラー           | `500` (`text/html`または`application/json`) | HTMLの場合は`document.write`を通じて完全な500エラーページをレンダリング（JSONの場合はエラーバナーを表示）                                        | HTMLの場合は500エラーページを表示（JSONの場合は`all`または`error`設定時にページをリロード）                           |
| 予期しない404エラー                | `404` (`text/html`または`application/json`) | HTMLの場合は対象を404エラーテンプレートに差し替え（JSONの場合はエラーバナーを表示）                                                              | `all`または`error`が設定されている場合はページをリロード（`success`指定時はビューを保持）                             |
