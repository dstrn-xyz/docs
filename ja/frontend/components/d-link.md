# d-link

クリックをインターセプトし非同期でコンテンツを取得してページ全体の再読み込みなしでDOMの差し替えを行うクライアント側ナビゲーションリンクコンポーネントおよびディレクティブです。

```html
<d-link href="/dashboard" class="nav-link">dashboard</d-link>

<a d-link href="/settings" target="#settings-panel" mode="inner">settings</a>
```

<a name="属性とプロパティ（d-link）"></a>

## 属性とプロパティ（d-link）

| 属性              | プロパティ       | 型        | デフォルト値 | フォーム | 説明                                                                      |
| :---------------- | :--------------- | :-------- | :----------- | :------- | :------------------------------------------------------------------------ |
| `href`            | `href`           | `string`  | `""`         | いいえ   | 遷移先URL                                                                 |
| `target`          | `target`         | `string`  | `""`         | いいえ   | 現在のページで差し替える要素のカンマ区切りCSSセレクター                   |
| `src`             | `src`            | `string`  | `""`         | いいえ   | 取得先ページから抽出する要素のカンマ区切りCSSセレクター                   |
| `mode`            | `mode`           | `string`  | `"inner"`    | いいえ   | コンテンツ差し替えモード（`"inner"`、`"outer"`、`"append"`、`"prepend"`） |
| `preserve-scroll` | `preserveScroll` | `boolean` | `false`      | いいえ   | 遷移後もウィンドウのスクロール位置を維持                                  |
| `d-full-reload`   | `dFullReload`    | `boolean` | `false`      | いいえ   | クライアント側ルーターをバイパスし標準のブラウザーリロードを実行          |

<a name="プログラム用api（d-link）"></a>

## プログラム用api（d-link）

```javascript
await dSPA.navigate('/dashboard', {
  targets: ['#content'],
  mode: 'inner',
  preserveScroll: true
});
```

<a name="動作仕様（d-link）"></a>

## 動作仕様（d-link）

- `<d-link>`要素および`<a d-link>`アンカーに対するクリックイベントをインターセプトします。
- 修飾キー（cmd、ctrl、shift、alt）を押しながらのクリック、`target="_blank"`、ハッシュリンク、または外部プロトコルURL（mailto、tel、javascript）は無視します。
- フェッチリクエストを開始する前に同一オリジンであることを検証します。
- ネットワークフェッチが失敗した場合はブラウザーの完全なナビゲーションにフォールバックします。
