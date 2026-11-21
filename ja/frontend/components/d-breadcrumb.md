# d-breadcrumb

宣言的なリンク子要素、カスタム文字またはアイコン区切り、`items`配列指定、および中間項目の折りたたみ展開を備えたパンくずリストコンポーネントです。

```html
<d-breadcrumb>
  <a d-link href="/">home</a>
  <a d-link href="/components">components</a>
  <span>breadcrumb</span>
</d-breadcrumb>

<d-breadcrumb separator="/" max-items="3">
  <a d-link href="/">home</a>
  <a d-link href="/workspace">workspace</a>
  <a d-link href="/projects">projects</a>
  <span>settings</span>
</d-breadcrumb>
```

<a name="属性とプロパティd-breadcrumb"></a>
## 属性とプロパティ（d-breadcrumb）

| 属性        | プロパティ  | 型       | デフォルト値 | フォーム | 説明                                                                          |
| :---------- | :---------- | :------- | :----------- | :------- | :---------------------------------------------------------------------------- |
| `separator` | `separator` | `string` | `""`         | いいえ   | カスタム区切り文字（`"/"`等）またはアイコンクラス（`dstrn-*`）。未指定時はシェブロン |
| `max-items` | `maxItems`  | `number` | `0`          | いいえ   | 項目数が上限を超えた際に中間項目を省略ボタンに折りたたむ上限数                |
| `items`     | `items`     | `array`  | `null`       | いいえ   | `{ label, href, link }`オブジェクト配列によるプログラム指定                   |

<a name="プログラム用apid-breadcrumb"></a>
## プログラム用api（d-breadcrumb）

```javascript
const trail = select('d-breadcrumb');

trail.expand();
trail.collapse();
trail.items = [
  { label: 'home', href: '/' },
  { label: 'docs', href: '/docs' },
  { label: 'breadcrumb' }
];
```

<a name="イベントd-breadcrumb"></a>
## イベント（d-breadcrumb）

| イベント   | 詳細 | バブリング | 説明                                       |
| :--------- | :--- | :--------- | :----------------------------------------- |
| `expand`   | なし | はい       | 折りたたまれた中間項目が展開されたときに送出 |
| `collapse` | なし | はい       | `collapse()`で再折りたたみされたときに送出 |

<a name="動作仕様d-breadcrumb"></a>
## 動作仕様（d-breadcrumb）

- 子要素（`<a d-link>`、`<d-link>`、ボタン、ネストされたポップオーバートリガー等）のライブDOMノードをHTML再解析せずそのまま保持します。
- 末尾の項目を現在のページとして自動強調し、RTLレイアウト時は区切りアイコンを反転表示します。
- 省略ボタンをクリックすると隠れた中間項目を一覧表示するフローティングポップオーバーが開きます。
