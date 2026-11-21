# d-buttongroup

隣接するボタン、テキストアドオン、入力欄、ポップオーバーやツールチップのトリガーを共通の外周角丸と統一ボーダーで結合する水平または垂直ボタングループコンテナです。

```html
<d-buttongroup>
  <button class="btn btn-outline">archive</button>
  <button class="btn btn-outline">report</button>
  <button class="btn btn-outline">snooze</button>
</d-buttongroup>

<d-buttongroup>
  <button class="btn btn-gray">copy</button>
  <span class="d-btngroup-sep"></span>
  <button class="btn btn-gray">paste</button>
</d-buttongroup>

<d-buttongroup orientation="vertical">
  <button class="btn btn-outline"><i class="dstrn-add-line"></i></button>
  <button class="btn btn-outline"><i class="dstrn-subtract-line"></i></button>
</d-buttongroup>

<d-buttongroup>
  <span class="d-btngroup-text">https://</span>
  <d-text-input placeholder="example.com"></d-text-input>
  <button class="btn btn-outline">go</button>
</d-buttongroup>
```

<a name="属性とプロパティd-buttongroup"></a>
## 属性とプロパティ（d-buttongroup）

| 属性          | プロパティ    | 型        | デフォルト値   | フォーム | 説明                                                           |
| :------------ | :------------ | :-------- | :------------- | :------- | :------------------------------------------------------------- |
| `orientation` | `orientation` | `string`  | `"horizontal"` | いいえ   | 配置方向（`"horizontal"`または`"vertical"`）                   |
| `disabled`    | `disabled`    | `boolean` | `false`        | いいえ   | グループ内の全コントロールを一括無効化（既存の無効状態は保持） |
| `dividers`    | `dividers`    | `boolean` | `false`        | いいえ   | 隣接する塗りつぶしボタン間に区切り線を自動表示                 |

<a name="動作仕様d-buttongroup"></a>
## 動作仕様（d-buttongroup）

- 重複ボーダーを自動統合し、論理角丸プロパティによってRTL環境でも正しく外周角丸を適用します。
- ネストされた`<d-buttongroup>`を自動検出し、グループ同士を結合せず適切な間隔のツールバーとして配置します。
- `<d-popover>`、`<d-tooltip>`、`<d-text-input>`、`<d-combobox>`、`.d-btngroup-text`、`.d-btngroup-sep`および`<hr>`区切り線とシームレスに組み合わせられます。
