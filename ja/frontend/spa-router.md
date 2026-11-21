# spaルーター

- [spaルーター](#spaルーター)
  - [はじめに](#はじめに)
  - [動作の仕組み](#動作の仕組み)
  - [デフォルトの差し替えターゲット](#デフォルトの差し替えターゲット)
  - [リンクとナビゲーション](#リンクとナビゲーション)
    - [アンカーのインターセプト](#アンカーのインターセプト)
    - [d-link要素](#d-link要素)
    - [d-linkでの完全なページ遷移の強制](#d-linkでの完全なページ遷移の強制)
  - [プログラムによるナビゲーション](#プログラムによるナビゲーション)
  - [差し替えモード](#差し替えモード)
  - [トランジションフック](#トランジションフック)
    - [グローバルトランジション](#グローバルトランジション)
    - [ナビゲーションごとのトランジション](#ナビゲーションごとのトランジション)
    - [フックリファレンス](#フックリファレンス)
  - [フォーム](#フォーム)
    - [基本的な使用法](#基本的な使用法)
    - [バリデーション](#バリデーション)
    - [jsonレスポンス](#jsonレスポンス)
    - [htmlレスポンス](#htmlレスポンス)
    - [リダイレクト処理](#リダイレクト処理)
    - [強制リロード](#強制リロード)
    - [コントローラーレスポンスのライフサイクル](#コントローラーレスポンスのライフサイクル)
    - [コールバック](#コールバック)
    - [プログラムによる送信](#プログラムによる送信)
  - [スクリプトの実行](#スクリプトの実行)
    - [スコープ付きスクリプト](#スコープ付きスクリプト)
    - [初期ページロードと永続スクリプト](#初期ページロードと永続スクリプト)
    - [モジュールスクリプト](#モジュールスクリプト)
    - [外部スクリプト](#外部スクリプト)
    - [永続要素（d-spa-keep）](#永続要素d-spa-keep)
  - [アクティブナビゲーション](#アクティブナビゲーション)
  - [csrfトークン管理](#csrfトークン管理)
  - [domモーフィング](#domモーフィング)
  - [スコープライフサイクル](#スコープライフサイクル)
  - [スクロールの挙動](#スクロールの挙動)
  - [デバッグモード](#デバッグモード)

<a name="はじめに"></a>

## はじめに

dframeworkにはdSPAと呼ばれる完全に統合されたクライアント側ルーターが含まれています。javascriptフレームワーク、ビルドステップ、または設定を必要とせずに、サーバーレンダリングされたアプリケーションをシングルページアプリケーションに変換します。`<a>`タグは`d-link`属性が付与されている場合にのみインターセプトされます。`<d-link>`カスタム要素は常にインターセプトされます。ルーターは遷移先を取得し、ページの残りの部分、レイアウトスクリプト、ソケット接続、音声再生を維持しながら、DOMの対象部分のみを差し替えます。

ルーターはアプリケーションにフレームワークのフロントエンドランタイムが含まれている場合に自動的に読み込まれます。インストールやインポートは不要です。

<a name="動作の仕組み"></a>

## 動作の仕組み

ユーザーがリンクをクリックすると、ルーターはバックグラウンドで遷移先ページをHTMLとして取得します。その後レスポンスを仮想ドキュメントとして解析し、設定された差し替えターゲットセレクター内のコンテンツを抽出し、現在のページの該当領域のみを的確に置き換えます。ブラウザーのURLと履歴は`pushState`を介して更新され、ドキュメントタイトルが同期され、CSRFトークンが更新され、新しいコンテンツ内のスクリプトが分離されたスコープ内で実行されます。

ブラウザーの戻るおよび進むナビゲーションは完全に処理されます。ユーザーが戻るボタンを押すと、ルーターは前のURLを取得し順方向ナビゲーションとまったく同様にコンテンツを差し替えます。

<a name="デフォルトの差し替えターゲット"></a>

## デフォルトの差し替えターゲット

デフォルトでは、ルーターは`dSPA.targets`および`window.dSPA_DEFAULT_TARGETS`を動的に確認し、候補セレクター`['#spa-container', 'main', 'body']`にフォールバックします。ルーターは現在のページと取得したページの両方に存在する最初のセレクターを解決します。`dSPA.targets`または`window.dSPA_DEFAULT_TARGETS`を設定することでこれをグローバルに上書きできます。

```html
<script defer>
  dSPA.targets = ['main'];
</script>
```

<a name="リンクとナビゲーション"></a>

## リンクとナビゲーション

<a name="アンカーのインターセプト"></a>

### アンカーのインターセプト

標準の`<a>`タグは`d-link`属性が付与されている場合にのみインターセプトされます。クリックされるとルーターはhrefを取得しデフォルトの対象領域を差し替えます。

```html
<a href="/dashboard" d-link>dashboard</a>
```

ルーターは以下の場合にインターセプトをスキップします:

- リンクが`d-link`属性を持たない標準の`<a>`である場合
- リンクが異なるオリジンを指している場合
- リンクに`target="_blank"`が設定されている場合
- リンクが`#`、`mailto:`、`tel:`、または`javascript:`で始まる場合
- ユーザーが修飾キー（ctrl、meta、shift、alt）を押している場合

<a name="d-link要素"></a>

### d-link要素

`<d-link>`はインターセプトされた`<a>`タグと同一に動作するカスタム要素ですが、その属性を通じてターゲット指定および差し替え動作をさらに細かく制御できます。

```html
<d-link href="/dashboard">dashboard</d-link>
```

ページのどの部分を置き換えるかは`target`属性を使用して指定できます。

```html
<d-link href="/admin/users" target="#spa-container">users</d-link>
```

取得したページでコンテンツ領域に異なるセレクターが使用されている場合、`src`属性を使用してレスポンスから抽出する要素を指定できます。

```html
<d-link href="/settings" target="#panel" src="#settings-panel">settings</d-link>
```

`mode`属性は新しいコンテンツをターゲット要素にどのように配置するかを制御します。

```html
<d-link href="/notifications" target="#feed" mode="prepend">load new</d-link>
```

ナビゲーション後もスクロール位置を維持するには`preserve-scroll`属性を追加します。

```html
<d-link href="/feed?page=2" target="#feed" preserve-scroll>next page</d-link>
```

<a name="d-linkでの完全なページ遷移の強制"></a>

### d-linkでの完全なページ遷移の強制

`d-link`のない標準の`<a>`タグはすでに通常のページナビゲーションを実行します。`<d-link>`要素で強制的にページ全体の再読み込みを行うには`d-full-reload`属性を使用します。

```html
<d-link href="/logout" d-full-reload>logout</d-link>
```

<a name="プログラムによるナビゲーション"></a>

## プログラムによるナビゲーション

`dSPA.navigate()`を使用してjavascriptからSPAナビゲーションをトリガーできます。これはリンクをクリックするのと同じフェッチ、差し替え、履歴更新のサイクルを実行します。

```javascript
dSPA.navigate('/dashboard');
```

オプションオブジェクトを渡してナビゲーションのターゲット、ソース、モード、トランジションを制御できます。

```javascript
dSPA.navigate('/home', {
  targets: ['#tabContent'],
  transitions: {
    beforeSwapTarget(oldEl, meta) {
      oldEl.style.opacity = '0';
    },
    afterSwapTarget(newEl, meta) {
      newEl.style.opacity = '1';
    }
  }
});
```

利用可能なオプションは以下のとおりです:

- `targets`: 現在のページで置き換える要素のCSSセレクター配列
- `srcTargets`: 取得したページから抽出する要素のCSSセレクター配列（デフォルトは`targets`）
- `mode`: 差し替えモード（`inner`、`outer`、`append`、`prepend`）
- `preserveScroll`: boolean、差し替え後もスクロール位置を維持
- `transitions`: トランジションフック関数を含むオブジェクト
- `addToHistory`: boolean、新しい履歴エントリを追加するかどうか（デフォルトは`true`）

<a name="差し替えモード"></a>

## 差し替えモード

ルーターはターゲット要素への新しいコンテンツの配置方法を制御する4つの差し替えモードをサポートしています。

**inner**（デフォルト）: ターゲット要素の子要素をクリアし新しいコンテンツの子要素を挿入します。ターゲット要素自体は保持されます。

**outer**: ターゲット要素自体を含めターゲット要素全体を新しいコンテンツ要素に置き換えます。

**append**: 既存の子要素を削除することなくターゲット要素の最後の子要素として新しいコンテンツを追加します。

**prepend**: 既存の子要素を削除することなくターゲット要素の最初の子要素として新しいコンテンツを挿入します。

<a name="トランジションフック"></a>

## トランジションフック

ルーターはナビゲーション中のコンテンツ遷移をアニメーション化できるライフサイクルフックを公開しています。フックは非同期処理に対応しています。ルーターは返されたプロミスが解決するまで待機してから処理を続行します。

<a name="グローバルトランジション"></a>

### グローバルトランジション

レイアウト内の`dSPA.transitions`にオブジェクトを代入することでグローバルトランジションフックを設定します。これらのフックはすべてのSPAナビゲーションで発火します。

```html
<script defer>
  const drawer = select('d-drawer');

  dSPA.transitions = {
    beforeSwap(oldNode, meta) {
      return drawer.close();
    },
    afterSwap(newNode, meta) {
      return drawer.open();
    },
  };
</script>
```

`beforeSwap`フックは古いコンテンツが削除される前に発火します。古いコンテンツの終了アニメーションに使用します。`afterSwap`フックは新しいコンテンツが挿入された後に発火します。新しいコンテンツの開始アニメーションに使用します。

<a name="ナビゲーションごとのトランジション"></a>

### ナビゲーションごとのトランジション

`dSPA.navigate()`に`transitions`オブジェクトを渡すことで単一のナビゲーションに対してグローバルフックを上書きできます。

```javascript
dSPA.navigate('/' + tab.dataset.tab, {
  targets: ['#tabContent'],
  transitions: {
    beforeSwapTarget(oldEl, meta) {
      oldEl.style.transform = 'translate3d(0, 5%, 0)';
      oldEl.style.opacity = '0';
    },
    afterSwapTarget(newEl, meta) {
      newEl.style.transform = 'translate3d(0, 0, 0)';
      newEl.style.opacity = '1';
    }
  }
});
```

<a name="フックリファレンス"></a>

### フックリファレンス

4つのフックが利用可能です。各フックは2つの引数を受け取ります（差し替え対象のDOM要素、および`url`、`link`、`src`、`dest`、`mode`を含む`meta`オブジェクト）。

- `beforeSwap(rootNode, meta)`: 差し替え開始前に一度発火しルートターゲット要素を受け取ります
- `beforeSwapTarget(oldNode, meta)`: 特定のターゲットが差し替えられる前にターゲットセレクターごとに一度発火します
- `afterSwap(rootNode, meta)`: 差し替えが完了しスクリプトが実行された後に一度発火し新しいルートコンテンツ要素を受け取ります
- `afterSwapTarget(newNode, meta)`: 特定のターゲットが差し替えられた後にターゲットセレクターごとに一度発火します

フックがプロミスを返した場合、ルーターはライフサイクルを続行する前にそれを直接awaitします。`dSPA.navigate()`に渡されたナビゲーションごとのトランジションはそのナビゲーションにおいてグローバルの`dSPA.transitions`を上書きします。

<a name="フォーム"></a>

## フォーム

<a name="基本的な使用法"></a>

### 基本的な使用法

`<d-form>`はページ全体の遷移ではなくfetch経由でフォームを送信するカスタム要素です。内部で標準の`<form>`要素をラップしそのsubmitイベントをインターセプトします。

```html
<d-form action="/register" method="POST" class="flex-column g-1">
  <input type="text" name="username" placeholder="username">
  <input type="password" name="password" placeholder="password">
  <button type="submit">register</button>
</d-form>
```

要素はすべてのリクエストにCSRFトークンを自動注入します。すでに`<d-form>`内に`<form>`要素がある場合要素はそれを直接使用します。省略した場合は`<d-form>`の`method`および`action`属性を使用して`<form>`が自動作成されます。

重複リクエストを防ぐため送信中はすべてのフォーム入力が無効化されます。

<a name="バリデーション"></a>

### バリデーション

`rules`属性を使用してクライアント側バリデーションルールを定義できます。構文はサーバー側のバリデーションルールを反映しています。

```html
<d-form action="/register" method="POST" rules='{"invite_token": "required"}'>
  <input type="text" name="invite_token" placeholder="invite token">
  <button type="submit">submit</button>
</d-form>
```

利用可能なクライアント側ルールは以下のとおりです: `required`、`string`、`number`、`integer`、`boolean`、`array`、`email`、`min:value`、`max:value`、`in:a,b,c`、`not_in:a,b,c`、`nullable`。

バリデーションが失敗するとフォームは最も近い`.input-wrapper`先祖（または入力自体）に`error`クラスを追加し直下に`error-message`要素を挿入します。エラーは次回の送信試行時に自動的にクリアされます。

<a name="jsonレスポンス"></a>

### jsonレスポンス

サーバーがJSONで応答した場合、フォームは特定のプロパティについてレスポンスボディを検査します。

レスポンスに`redirect`プロパティが含まれている場合、フォームはSPAルーターを介してそのURLに遷移します。レスポンスに`force: true`プロパティも含まれている場合は代わりにページ全体のリダイレクトを実行します。

レスポンスに`errors`オブジェクトが含まれている場合、フォームはクライアント側バリデーションと同じ規則に従って対応する各入力フィールドの横にエラーメッセージを表示します。

<a name="htmlレスポンス"></a>

### htmlレスポンス

サーバーがHTMLで応答した場合、フォームはその対象要素のコンテンツをレスポンスで置き換えます。対象のデフォルトは`body`ですが`target`属性を使用して上書きできます。

```html
<d-form action="/search" method="GET" target="#results">
  <input type="text" name="q" placeholder="search">
  <button type="submit">search</button>
</d-form>
```

`navigate`属性が存在する場合、フォームは単純なコンテンツ置換ではなくSPAルーターの完全なナビゲーションパイプラインを使用します。

```html
<d-form action="/search" method="GET" target="#results" navigate>
  <input type="text" name="q" placeholder="search">
</d-form>
```

<a name="リダイレクト処理"></a>

### リダイレクト処理

サーバーがHTTPリダイレクト（3xx）で応答した場合、フォームはSPAルーターを通じてそれに従い、新しいページコンテンツをデフォルトのターゲットに差し替えてブラウザーのURLを更新します。

<a name="強制リロード"></a>

### 強制リロード

`force-reload`属性はSPAルーターを通じてレスポンスを処理する代わりに特定の条件下でページ全体の再読み込みをトリガーするようフォームに指示します。

```html
<d-form action="/save" method="POST" force-reload>
<d-form action="/register" method="POST" force-reload="success">
<d-form action="/save" method="POST" force-reload="error">
```

有効な値は`true`（または同等である空文字列）、`all`、`success`、`error`です。

<a name="コントローラーレスポンスのライフサイクル"></a>

### コントローラーレスポンスのライフサイクル

以下の表は`<d-form>`が各サーバーコントローラーレスポンスをどのように処理するか、また`force-reload`がデフォルトの処理をどのように変更するかを示しています:

| コントローラーの戻り値 | ステータス / タイプ | デフォルトの動作（force-reloadなし） | force-reload指定時の動作 |
| :--------------------------------- | :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| `return json({ data })` | `200` (`application/json`) | ペイロードを解析し`d-success`と`d-response`を送出、`callback`が定義されている場合は実行、ページリロードなしでフォームを画面に残す | `force-reload`、`all`、または`success`が設定されている場合はページをリロード（`error`指定時は無視） |
| `return json({ redirect })` | `200` (`application/json`) | リロードなしで`dSPA.navigate(redirect)`を通じて遷移（`force: true`または`X-Redirect-Force`が設定されている場合は完全なページリダイレクトを実行） | SPAルーター経由で遷移（`force: true`の場合はハードリダイレクト） |
| `return json().withErrors(errors)` | `422` (`application/json`) | 入力欄に`.error`を付与しフィールド直下に`.error-message`（または上部に`.error-banner`）を注入、`d-error`と`d-response`を送出 | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はインラインエラーを保持） |
| `validate()` 失敗 | `422` (`application/json`) | フィールドバリデーションエラーをインラインでレンダリング、`d-error`と`d-response`を送出、フォーム状態を維持 | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はインラインエラーを保持） |
| `return abort(status, message)` | `4xx`または`5xx` (`application/json`) | フォーム上部に`.error-banner`を表示、`d-error`と`d-response`を送出、フォーム入力を再度有効化 | `force-reload`、`all`、または`error`が設定されている場合はページをリロード（`success`指定時はバナーを表示） |
| `return render(view, data)` | `200` (`text/html`) | 返却されたマークアップを対象セレクター（デフォルトは`body`）に差し替え、スクリプトを実行、`d-success`を送出（`navigate`設定時はURLを更新） | `force-reload`、`all`、または`success`が設定されている場合はページをリロード（`error`指定時は差し替えを実行） |
| 予期しないサーバーエラー | `500` (`text/html`または`application/json`) | HTMLの場合は`document.write`を通じて完全な500エラーページをレンダリング（JSONの場合はエラーバナーを表示） | HTMLの場合は500エラーページを表示（JSONの場合は`all`または`error`設定時にページをリロード） |
| 予期しない404エラー | `404` (`text/html`または`application/json`) | HTMLの場合は対象を404エラーテンプレートに差し替え（JSONの場合はエラーバナーを表示） | `all`または`error`が設定されている場合はページをリロード（`success`指定時はビューを保持） |

<a name="コールバック"></a>

### コールバック

`callback`属性に関数名を指定することでレスポンス成功後にjavascript関数を実行できます。関数は解析されたレスポンスデータを引数として受け取ります。

```html
<d-form action="/api/magic" method="POST" callback="onMagicSuccess">
  <button type="submit">generate</button>
</d-form>

<script>
  function onMagicSuccess(data) {
  }
</script>
```

<a name="プログラムによる送信"></a>

### プログラムによる送信

`<d-form>`要素上で`.submit()`を呼び出すことでプログラムからフォームを送信できます。

```javascript
select('d-form').submit();
```

また`.serialize()`を呼び出すことで現在のフォームデータを`FormData`オブジェクトとして取得できます。

```javascript
const data = select('d-form').serialize();
```

<a name="スクリプトの実行"></a>

## スクリプトの実行

ルーターは各ナビゲーション後に差し替えられたコンテンツ内にあるすべての`<script>`タグを実行します。すべてのスクリプトはタイマー、リスナー、fetch、アニメーションフレームのライフサイクル認識置換を提供する分離されたスコープ内で実行されます。後続のナビゲーション中にスクリプトを所有するコンテンツがDOMから削除されると、そのタイマー、リスナー、保留中のリクエストはすべて自動的にクリーンアップされます。

<a name="スコープ付きスクリプト"></a>

### スコープ付きスクリプト

差し替えられたコンテンツ内のインラインスクリプトは標準ブラウザーAPIのスコープ付きバージョンをローカル変数として受け取ります。グローバルAPIとまったく同じように使用できますがコンテンツから離れるナビゲーションが発生したときに自動的に破棄されます。

すべてのインラインスクリプト内で以下のスコープ付きAPIが利用可能です:

- `setTimeout`、`setInterval`、`clearTimeout`、`clearInterval`
- `requestAnimationFrame`、`cancelAnimationFrame`
- `fetch`
- `listen(target, event, handler, options)`: スコープ付きイベントリスナー（解除用関数を返却）
- `listenAll(targets, event, handler, options)`: 複数要素に対するスコープ付きイベントリスナー（複合解除用関数を返却）
- `unlisten(target, event, handler, options)`: イベントリスナーの手動削除
- `unlistenAll(targets, event, handler, options)`: 複数要素に対するイベントリスナーの手動削除
- `nextFrame()`: 次のアニメーションフレームで解決するプロミスを返却
- `sleep(ms)`: 指定された遅延後に解決するプロミスを返却
- `dSPA_SCOPE`: 高度な用途向けの生スコープオブジェクト
- `Socket`: フレームワークのソケットファサード
- `dSPA`: ルーター自体

```html
<div id="counter">0</div>
<script>
  let count = 0;
  const el = document.querySelector('#counter');

  setInterval(() => {
    count++;
    el.textContent = count;
  }, 1000);
</script>
```

ユーザーがページから離れるとインターバルは自動的にクリアされます。手動のクリーンアップは不要です。

<a name="初期ページロードと永続スクリプト"></a>

### 初期ページロードと永続スクリプト

標準のページビューおよび差し替えられたコンテンツでは通常の`<script>`タグを記述できます。差し替え領域内のすべてのインラインスクリプトおよび外部スクリプトはルーターによって自動的に管理およびスコープ化されます。

> [!TIP]
> 初回のハードページロード時において管理スコープとアクティブなWebSocket接続へのアクセスが必要なレイアウトスクリプトには`type="text/dspa"`を使用します。dSPA経由で遷移した通常のビューは管理スコープ内で自動的にスクリプトを実行します。

```html
<script type="text/dspa">
  Socket.on('update:level', (data) => {
    selectAll('[data-level]').forEach(el => {
      el.textContent = data.level;
    });
  });
</script>
```

ブラウザーパーサーは通常のページ読み込み時に`type="text/dspa"`を無視します。`DOMContentLoaded`時かつ`Socket.connect()`の完了後、ルーターは初回管理スコープ内ですべての`type="text/dspa"`スクリプトを実行し、`Socket`、`listen`、自動クリアタイマーなどのスコープ付きユーティリティを提供します。

dSPA経由でナビゲートされる通常のビューおよび差し替えコンテンツでは標準の`<script>`タグがすでに管理スコープ内で自動実行されるため`type="text/dspa"`は不要です。

<a name="モジュールスクリプト"></a>

### モジュールスクリプト

ESモジュールスクリプト（`type="module"`）がサポートされています。デフォルトではモジュールスコープの分離によりクラシックスクリプトで使用されるラッピング手法が防止されるため、モジュールスクリプトはスコープ付きAPI変数を受け取りません。

モジュールスクリプトでスコープ注入を有効にするには`d-spa-scope`属性を追加します。

```html
<script type="module" d-spa-scope>
  const el = document.querySelector('#timer');
  setInterval(() => {
    el.textContent = new Date().toLocaleTimeString();
  }, 1000);
</script>
```

<a name="外部スクリプト"></a>

### 外部スクリプト

差し替えられたコンテンツ内の外部スクリプト（`src`属性を持つスクリプト）はドキュメント順に従ってすべてのSPAナビゲーションで自動的に再実行されます。ルーターは後続のページスクリプトを実行する前にそれらを読み込んで実行します。

SPAナビゲーション時にルーターが特定のスクリプトタグを実行しないようにするには`d-spa-ignore`属性を追加します。

```html
<script src="/analytics.js" d-spa-ignore></script>
```

<a name="永続要素（d-spa-keep）"></a>

### 永続要素（d-spa-keep）

bodyの差し替えをまたいで保持する必要がある要素やスクリプト（開発ツールや永続的なレイアウトステートなど）には`d-spa-keep`属性を使用します。bodyの差し替えが発生した際、`d-spa-keep`を持つすべての要素が保持され新しいbodyに再アタッチされます。

```html
<div id="player-container" d-spa-keep>
</div>
```

<a name="アクティブナビゲーション"></a>

## アクティブナビゲーション

ルーターは現在アクティブなナビゲーションリンクをハイライトする宣言的な方法を提供します。任意の要素に`d-nav-active`属性を追加して現在のURLに基づいてどのCSSクラスを切り替えるかを指定します。

```html
<d-link
  href="/home"
  d-nav-url="/home"
  d-nav-active="d-link-active"
  class="text-content bdr hover:bg-container-l tr-03 px-1 py-075 flex-center">
  <i class="dstrn-home"></i>
</d-link>
```

`d-nav-url`はこの要素が対応するURLパスを指定します。`href`を持つ要素（`<d-link>`や`<a d-link>`など）で省略された場合、ルーターは`href`の読み取りにフォールバックします。先頭と末尾のスラッシュは自動的に正規化されるため、`/library/`、`library`、`/library`はすべて一貫して一致します。

デフォルトの一致判定はパスセグメントに基づくプレフィックス一致です。`/home`の`d-nav-url`は`/home`および`/home/settings`に一致し`/home-feed`は無視します。完全一致を要求するには`d-nav-exact`属性を追加します。

アクティブ状態はすべてのSPAナビゲーション後に自動的に再計算されます。

<a name="csrfトークン管理"></a>

## csrfトークン管理

ルーターはナビゲーション間でCSRFトークンを自動管理します。ページがフェッチされるとルーターはレスポンスから`csrf-token`および`shield-challenge`メタタグを抽出し現在のページのメタタグを新しい値で更新します。

同一オリジンを対象とするすべての変更系fetchリクエスト（`POST`、`PUT`、`PATCH`、`DELETE`）には`X-CSRF-TOKEN`ヘッダーが自動注入されます。これはグローバルの`fetch()`関数とインラインスクリプトに提供されるスコープ付き`fetch()`の両方に適用されます。クライアント側のリクエストに手動でCSRFトークンを添付する必要は一切ありません。

<a name="domモーフィング"></a>

## domモーフィング

差し替え中、ルーターは古いDOMを新しいDOMでやみくもに置き換えることはしません。両方のツリーを走査し古いツリーを新しいツリーに変換するために必要な最小限の変更を適用するモーフィングアルゴリズムを使用します。これによりフォーカス状態、入力値、ネストされたコンテナー内のスクロール位置、および変更されなかったDOMノードに紐づく実行時ステートが保持されます。

モーフィングアルゴリズムは要素属性の更新、テキストノードの変更、IDによる子の並べ替え、input/select/textarea値の同期を処理します。

`customElements.define`で登録されたカスタム要素はモーフィング後に適切にアップグレードされます。カスタム要素が検出された場合、古い要素は新しい要素に完全に置き換えられ`customElements.upgrade()`を介してアップグレードされます。

<a name="スコープライフサイクル"></a>

## スコープライフサイクル

ルーターは差し替えられたコンテンツ内のスクリプトを実行するたびにスコープを作成します。スコープはそれらのスクリプトによって作成されたすべてのタイマー、アニメーションフレーム、fetchリクエスト、イベントリスナーを所有します。スコープを所有するDOMノードが削除されると（後続のナビゲーションまたは直接のDOM操作のいずれかによって）、`MutationObserver`が削除を検知し自動的にスコープを破棄してすべてのタイマーをクリアし保留中のすべてのfetchを中止しすべてのイベントリスナーを削除します。

これはページスクリプトでクリーンアップコードを記述する必要が一切ないことを意味します。ルーターが自動的に処理します。

<a name="スクロールの挙動"></a>

## スクロールの挙動

デフォルトでは、ルーターはすべてのナビゲーション後にページの最上部までスクロールします。遷移先URLに`#hash`フラグメントが含まれている場合（例えば`/profile#settings`や`{{ route('home') }}#native`）、ルーターは一致する要素を自動的に見つけてビューまでスクロールします。

現在のページ上の`#hash`を指すリンクをクリックした際（既に`/`にいるときの`/#layers`など）、ルーターはナビゲーションをインターセプトし不要なネットワークフェッチをスキップしてブラウザー履歴を更新し対象要素をスムーズにスクロール表示します。

> [!TIP]
> 永続レイアウト内のセクションリンクには相対ハッシュフラグメント（`#native`）の代わりにハッシュ付きの完全なルートURL（例: `{{ route('home') }}#native`）を使用することで任意のページからナビゲーションが機能します。

`<d-link>`に`preserve-scroll`属性を追加するか`dSPA.navigate()`に`preserveScroll: true`を渡すことでナビゲーションごとにスクロールを無効化できます。

<a name="デバッグモード"></a>

## デバッグモード

`local`環境ではフレームワークはルーターのデバッグモードを自動的に有効にします。デバッグモードでは各ナビゲーション、差し替えのタイミング、スコープの作成とクリーンアップ、潜在的なメモリリークに関する詳細情報が記録されます。

ルーターは各スコープが存続したナビゲーションの回数を追跡します。クリーンアップされずにスコープが5回を超えるナビゲーションを存続した場合、潜在的なメモリリークを示す警告がログに出力されます。孤立したスコープ（どのDOMノードにもリンクされていないスコープ）も報告されます。

デバッグモードは手動で切り替えることができます。

```javascript
dSPA.debug = true;
```
