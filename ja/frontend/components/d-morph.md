# d-morph

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/morph.mp4" type="video/mp4">
  </video>
</p>

ドキュメント内の任意の位置にある名前付き要素間で連続的なジオメトリ、border-radius、box-shadow、およびフォーム状態の補間を伴う遷移を行う物理ベースのモーフィングアニメーションエンジン（`dMotion`）です。

```html
<d-morph name="summary-card" d-action="click" d-to="detail-card" d-duration="350" d-visible>
  <div class="card p-2">
    <h4>project summary</h4>
    <p>click to expand details</p>
  </div>
</d-morph>

<d-morph name="detail-card" d-action="click" d-to="summary-card" d-duration="350">
  <div class="card p-4">
    <h4>detailed project view</h4>
    <p>full metadata and extended controls</p>
  </div>
</d-morph>
```

## 属性とプロパティ

| 属性         | プロパティ | 型        | デフォルト値 | 説明                                                           |
| :----------- | :--------- | :-------- | :----------- | :------------------------------------------------------------- |
| `name`       | `name`     | `string`  | `""`         | `window.dMotion`に登録される一意のモーフ識別子                 |
| `d-visible`  | なし       | `boolean` | `false`      | このモーフ要素を初期状態でアクティブかつ可視としてマーク       |
| `d-action`   | なし       | `string`  | `""`         | トリガーアクション（`"click"`、`"long-press"`、`"click-out"`） |
| `d-to`       | なし       | `string`  | `""`         | 遷移先の対象モーフ名                                           |
| `d-duration` | なし       | `number`  | 計算値       | ミリ秒単位の遷移時間（省略時は距離から計算）                   |

## プログラム用api（window.dMotion）

```javascript
const morph = dMotion.get('summary-card');
const prev = dMotion.getPrevious();

await dMotion.transition('summary-card', 'detail-card', optionalTriggerNode);

dMotion.listen('summary-card', 'detail-card', () => {
  console.log('transition starting');
}, 'before');

dMotion.unlisten('summary-card', 'detail-card', callback, 'before');

morph.show();
morph.hide();
```

## 動作仕様

- ページ上の任意の2つの要素間で滑らかなアニメーションモーフィング遷移を作成します。
- アニメーション中も視覚スタイル（サイズ、位置、境界の丸み、シャドウ）と要素の状態を維持します。
- クリック、長押し、外部クリックのトリガーをサポートします。
- `d-no-morph`を持つ子要素は遷移をトリガーすることなく操作可能です。
