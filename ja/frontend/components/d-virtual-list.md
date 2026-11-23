# d-virtual-list

CSSのcontent-visibilityと最下部近接検知を利用し、画面外要素のレイアウトおよび描画処理をスキップする軽量DOM仮想化コンポーネントです。

```html
<d-virtual-list item-size="64px" buffer="800px 0px 800px 0px" threshold="600">
  @foreach ($items as $item)
    <div class="p-3 bg-container mb-2 rounded">{{ $item->title }}</div>
  @endforeach
</d-virtual-list>
```

<a name="属性とプロパティ-d-virtual-list"></a>
## 属性とプロパティ

| 属性            | プロパティ     | 型       | デフォルト値            | フォーム | 説明                                                           |
| :-------------- | :------------- | :------- | :---------------------- | :------- | :------------------------------------------------------------- |
| `item-size`     | `itemSize`     | `string` | `"52px"`                | いいえ   | contain-intrinsic-size用の項目推定高さフォールバック           |
| `buffer`        | `buffer`       | `string` | `"600px 0px 600px 0px"` | いいえ   | 画面外項目の先行レンダリングを行うIntersectionObserverのマージン |
| `threshold`     | `threshold`    | `number` | `500`                   | いいえ   | reachbottomイベントを発火する最下部からのピクセル距離           |
| `cooldown`      | `cooldown`     | `number` | `250`                   | いいえ   | reachbottomイベント連続発火のクールダウンミリ秒                |
| `scroll-parent` | `scrollParent` | `string` | `""`                    | いいえ   | 任意のスクロールコンテナを指定するCSSセレクタ                  |

<a name="プログラム用api-d-virtual-list"></a>
## プログラム用api

```javascript
const list = select('d-virtual-list');

list.itemSize = '72px';
list.threshold = 400;

// 最下部への近接判定を手動実行
list.checkBottom();
```

<a name="イベント-d-virtual-list"></a>
## イベント

| イベント      | 詳細                      | バブリング | 説明                                           |
| :------------ | :------------------------ | :--------- | :--------------------------------------------- |
| `reachbottom` | `{ distance, threshold }` | はい       | リストの最下部付近までスクロールしたときに送出 |

<a name="動作仕様-d-virtual-list"></a>
## 動作仕様

- ライトDOMツリーを維持したまま、画面外項目に`content-visibility: hidden`と`contain-intrinsic-size`を適用して描画負荷を低減します。
- 親階層、`scroll-parent`セレクタ、またはウィンドウのビューポートからスクロールコンテナを自動判別します。
- 末尾アンカー要素とIntersectionObserverを連携させ、無限スクロール用の`reachbottom`イベントを安定して検知します。
- コンポーネントが画面外に移動した際は自動的に判定処理を一時停止します。