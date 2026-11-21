# d-combobox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/combobox.mp4" type="video/mp4">
  </video>
</p>

自動フローティング配置、キーボードナビゲーション、動的な選択肢注入を備えた検索可能で拡張可能なセレクトコンボボックスです。

```html
<d-combobox
  placeholder="select department"
  allow-search
  allow-input
  name="department">
  <option value="eng" selected>engineering</option>
  <option value="des">product design</option>
  <option value="ops">operations</option>
</d-combobox>
```

## 属性とプロパティ

| 属性           | プロパティ     | 型        | デフォルト値   | フォーム | 説明                                                                                       |
| :------------- | :------------- | :-------- | :------------- | :------- | :----------------------------------------------------------------------------------------- |
| `placeholder`  | `placeholder`  | `string`  | `""`           | いいえ   | 選択肢が選択されていないときに表示されるプレースホルダー文字列                             |
| `value`        | `value`        | `string`  | `null`         | はい     | 現在選択されている選択肢の値                                                               |
| `allow-search` | `allowSearch`  | `boolean` | `false`        | いいえ   | リアルタイム検索フィルタリング入力を有効化                                                 |
| `allow-input`  | `allowInput`   | `boolean` | `false`        | いいえ   | 入力とプラスボタンによるカスタム選択肢の追加を許可                                         |
| `horizontal`   | `isHorizontal` | `boolean` | `false`        | いいえ   | 選択肢のレイアウトを垂直リストから水平チップに切り替え                                     |
| `empty-text`   | `emptyText`    | `string`  | `"no results"` | いいえ   | 一致する選択肢がない場合に表示されるカスタム空状態メッセージ                               |
| `name`         | `name`         | `string`  | `""`           | いいえ   | フォームフィールド名                                                                       |
| `id`           | `id`           | `string`  | `""`           | いいえ   | コンポーネントID                                                                           |
| `options`      | `options`      | `array`   | `[]`           | いいえ   | `{value, text, content, html}`オブジェクトの配列（リッチコンテンツには`html: true`を設定） |

## プログラム用api

```javascript
const combo = select('d-combobox');

combo.addOption('mktg', 'marketing');
combo.removeOption('ops');
combo.clearOptions();
combo.value = 'eng';
dCombobox.closeAll();
```

## イベント

| イベント | 詳細         | バブリング | 説明                                   |
| :------- | :----------- | :--------- | :------------------------------------- |
| `change` | 選択値文字列 | はい       | 選択肢が選択または追加されたときに送出 |
| `open`   | なし         | はい       | ドロップダウンパネルが開いたときに送出 |
| `close`  | なし         | はい       | ドロップダウンパネルが閉じたときに送出 |

## 動作仕様

- HTML内で宣言された子`<option>`タグを自動的に解析します。
- 画面からはみ出すことなく上下に開くようビューポートの空き領域を自動計算します。
- スクロール中やリサイズ中もフローティング座標の配置を維持します。
- 完全なキーボードナビゲーションを提供します（矢印キー、Home、End、Enter、Escape）。
- 検索クエリに一致する項目がない場合に空状態インジケーターを表示します。
