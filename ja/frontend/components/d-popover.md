# d-popover

ネイティブの`popover="manual"`トップレイヤーと単一パス`requestAnimationFrame`によるGPU配置を備えたフローティングポップオーバーコンポーネントです。クリック、ホバー、手動トリガー、自動反転、ネストされたスクロール追従、およびネストポップオーバーに対応しています。

```html
<d-popover side="bottom" align="start">
  <button slot="trigger" class="btn btn-outline">open popover</button>
  <div slot="content">
    <div class="d-popover-header">
      <h4 class="d-popover-title">dimensions</h4>
      <p class="d-popover-desc">set the dimensions for the layer.</p>
    </div>
    <d-text-input placeholder="width" value="100%"></d-text-input>
  </div>
</d-popover>

<d-popover trigger="hover" side="top" align="center">
  <button slot="trigger" class="btn btn-gray">hover reveal</button>
  <div slot="content">popover content</div>
</d-popover>
```

<a name="属性とプロパティd-popover"></a>
## 属性とプロパティ（d-popover）

| 属性                     | プロパティ            | 型        | デフォルト値 | フォーム | 説明                                                                           |
| :----------------------- | :-------------------- | :-------- | :----------- | :------- | :----------------------------------------------------------------------------- |
| `opened`                 | `opened`              | `boolean` | `false`      | いいえ   | 開閉状態                                                                       |
| `trigger`                | `trigger`             | `string`  | `"click"`    | いいえ   | トリガー方式（`"click"`、`"hover"`、`"manual"`）                               |
| `side`                   | `side`                | `string`  | `"bottom"`   | いいえ   | 優先表示位置（`"bottom"`、`"top"`、`"left"`、`"right"`、`"inline-start"`、`"inline-end"`） |
| `align`                  | `align`               | `string`  | `"start"`    | いいえ   | 軸方向の整列（`"start"`、`"center"`、`"end"`）                                 |
| `offset`                 | `offset`              | `number`  | `8`          | いいえ   | アンカー要素との間隔（ピクセル）                                               |
| `match-width`            | `matchWidth`          | `boolean` | `false`      | いいえ   | ポップオーバー幅をアンカー要素の幅に一致させる                                 |
| `hover-delay`            | `hoverDelay`          | `number`  | `120`        | いいえ   | ホバー開閉時の遅延猶予時間（ミリ秒）                                           |
| `target`                 | `target`              | `string`  | `""`         | いいえ   | 外部アンカー要素のCSSセレクタ                                                  |
| `close-on-click-outside` | `closeOnClickOutside` | `boolean` | `true`       | いいえ   | 外部クリック時に自動で閉じる                                                   |
| `close-on-escape`        | `closeOnEscape`       | `boolean` | `true`       | いいえ   | `Escape`キー押下時に自動で閉じる                                               |
| `exclusive`              | `exclusive`           | `boolean` | `true`       | いいえ   | 開いた際に無関係な他ポップオーバーを閉じる（親ポップオーバーは保持）           |
| `unstyled`               | `unstyled`            | `boolean` | `false`      | いいえ   | デフォルトのカード背景・枠線・余白を外しカスタムオーバーレイとして利用する     |

<a name="プログラム用apid-popover"></a>
## プログラム用api（d-popover）

```javascript
const pop = select('d-popover');

pop.open();
pop.close();
pop.toggle();
pop.reposition();
pop.setAnchor(select('#custom-anchor'));
const anchor = pop.getAnchor();

dPopover.closeAll();
```

<a name="イベントd-popover"></a>
## イベント（d-popover）

| イベント | 詳細 | バブリング | 説明                             |
| :------- | :--- | :--------- | :------------------------------- |
| `open`   | なし | はい       | ポップオーバーが開いたときに送出 |
| `close`  | なし | はい       | ポップオーバーが閉じたときに送出 |

<a name="動作仕様d-popover"></a>
## 動作仕様（d-popover）

- ブラウザのトップレイヤー（`popover="manual"`）に表示されるため、`overflow: hidden`やスタッキングコンテキストによるクリッピングを受けず、ネストされたスクロールコンテナ内でも`translate3d`で滑らかに追従します。
- 表示領域が不足する場合は反対側へ自動反転し、アンカー側からスプリングアニメーションで展開します。
- `slot="trigger"` / `slot="content"`による明示指定、位置ベース指定（先頭子要素がトリガー）、および外部`target`セレクタ指定のすべてに対応しています。
