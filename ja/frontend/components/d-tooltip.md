# d-tooltip

要素がホバーまたはキーボードフォーカスを受け取ったときに関連情報を表示する軽量なポップアップです。トップレイヤーの`d-popover`配置エンジンを基盤とし、統合されたキーボードショートカット表示や無効化トリガーボタンへの安全な対応を備えています。

```html
<d-tooltip text="save document" shortcut="cmd+s" side="top">
  <button class="btn btn-outline">save</button>
</d-tooltip>

<d-tooltip text="action unavailable" side="right">
  <button class="btn btn-outline" disabled>disabled action</button>
</d-tooltip>

<d-tooltip>
  <button slot="trigger" class="btn btn-gray"><i class="dstrn-add-line"></i></button>
  <span slot="content">custom tooltip markup</span>
</d-tooltip>
```

<a name="属性とプロパティ-d-tooltip"></a>
## 属性とプロパティ

| 属性       | プロパティ | 型        | デフォルト値 | フォーム | 説明                                                                                                    |
| :--------- | :--------- | :-------- | :----------- | :------- | :------------------------------------------------------------------------------------------------------ |
| `text`     | `text`     | `string`  | `""`         | いいえ   | ツールチップのテキスト内容                                                                              |
| `shortcut` | `shortcut` | `string`  | `""`         | いいえ   | `<d-key>`によって自動表示されるキーボードショートカット（例: `"cmd+s"`, `"alt+left"`）                 |
| `side`     | `side`     | `string`  | `"top"`      | いいえ   | 推奨配置位置（`"top"`, `"bottom"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`）            |
| `align`    | `align`    | `string`  | `"center"`   | いいえ   | トリガー軸に沿った整列（`"start"`, `"center"`, `"end"`）                                                |
| `delay`    | `delay`    | `number`  | `150`        | いいえ   | 表示までのホバー猶予遅延ミリ秒                                                                          |
| `offset`   | `offset`   | `number`  | `6`          | いいえ   | トリガーとツールチップバブル間のピクセル間隔                                                            |
| `disabled` | `disabled` | `boolean` | `false`      | いいえ   | ツールチップの表示を無効化                                                                              |

<a name="プログラム用api-d-tooltip"></a>
## プログラム用api

```javascript
const tip = select('d-tooltip');

tip.open();
tip.close();
tip.toggle();
tip.setAnchor(select('#custom-anchor'));
const anchor = tip.getAnchor();
tip.text = 'new label';
tip.disabled = true;

// アクティブなすべてのツールチップを閉じる
dTooltip.closeAll();
```

<a name="イベント-d-tooltip"></a>
## イベント

| イベント | 詳細 | バブリング | 説明                             |
| :------- | :--- | :--------- | :------------------------------- |
| `open`   | なし | はい       | ツールチップが表示されたときに送出 |
| `close`  | なし | はい       | ツールチップが閉じたときに送出   |

<a name="動作仕様-d-tooltip"></a>
## 動作仕様

- `d-popover`を介してブラウザのトップレイヤーにレンダリングされるため、親要素の`overflow: hidden`によるクリッピングや`z-index`スタッキングコンテキストの影響を受けません。
- `shortcut`指定時は自動的にプラットフォーム対応の`<d-key>`バッジをレイアウトします。
- 外側ラッパー経由でポインターイベントを処理するため、無効化されたボタン上でも確実にツールチップを表示します。
- キーボード操作に対応し、`focusin`で表示され`focusout`で非表示になります。
