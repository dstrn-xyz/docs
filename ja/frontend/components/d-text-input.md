# d-text-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/input.mp4" type="video/mp4">
  </video>
</p>

アイコンプレフィックス、キーボードショートカットバインディング、数値ステッパーボタン、フォーム統合を備えた高機能入力フィールドです。

```html
<d-text-input
  placeholder="search records"
  icon="dstrn-search-line"
  shortcut="cmd+k"
  name="query">
</d-text-input>
```

## 属性とプロパティ

| 属性           | プロパティ     | 型        | デフォルト値 | フォーム | 説明                                                            |
| :------------- | :------------- | :-------- | :----------- | :------- | :-------------------------------------------------------------- |
| `placeholder`  | `placeholder`  | `string`  | `""`         | いいえ   | プレースホルダー文字列                                          |
| `icon`         | `icon`         | `string`  | `""`         | いいえ   | アイコンクラス名                                                |
| `type`         | `type`         | `string`  | `"text"`     | いいえ   | 入力タイプ（`"text"`、`"password"`、`"search"`、`"number"`）    |
| `autocomplete` | `autocomplete` | `boolean` | `false`      | いいえ   | ブラウザー自動補完フラグ                                        |
| `value`        | `value`        | `string`  | `""`         | はい     | 現在の入力文字列値                                              |
| `name`         | `name`         | `string`  | `""`         | いいえ   | フォームフィールド名                                            |
| `id`           | `id`           | `string`  | `""`         | いいえ   | コンポーネントID                                                |
| `shortcut`     | `shortcut`     | `string`  | `""`         | いいえ   | グローバルキーボードショートカット（例: `"cmd+k"`、`"ctrl+f"`） |
| `disabled`     | `disabled`     | `boolean` | `false`      | いいえ   | 入力を無効化                                                    |
| `readonly`     | `readonly`     | `boolean` | `false`      | いいえ   | 入力を読み取り専用モードに設定                                  |
| `min`          | `min`          | `string`  | `""`         | いいえ   | 最小数値                                                        |
| `max`          | `max`          | `string`  | `""`         | いいえ   | 最大数値                                                        |
| `step`         | `step`         | `string`  | `"1"`        | いいえ   | 数値コントロールのステップ値                                    |

## プログラム用api

```javascript
const input = select('d-text-input');

input.value = 'search term';
input.placeholder = 'type command';
input.disabled = false;
input.focus();
input.blur();
```

## イベント

| イベント | 詳細     | バブリング | 説明                       |
| :------- | :------- | :--------- | :------------------------- |
| `input`  | 文字列値 | はい       | キーストローク入力時に送出 |
| `change` | 文字列値 | はい       | 入力変更時に送出           |

## 動作仕様

- OSをまたいでフィールドに自動フォーカスするグローバルキーボードショートカット（例: `cmd+k`）をサポートします。
- `type="number"`設定時に増減ステッパーコントロールを表示します。
