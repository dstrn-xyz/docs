# d-checkbox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/checkbox.mp4" type="video/mp4">
  </video>
</p>

同期ラベルとネイティブフォーム送信統合を備えたフォーム関連のチェックボックススイッチです。

```html
<d-checkbox text="enable notifications" checked name="notifications" value="1"></d-checkbox>
<d-checkbox text="accept terms" placement="before" name="terms"></d-checkbox>
```

## 属性とプロパティ

| 属性        | プロパティ  | 型        | デフォルト値 | フォーム | 説明                                                               |
| :---------- | :---------- | :-------- | :----------- | :------- | :----------------------------------------------------------------- |
| `checked`   | `checked`   | `boolean` | `false`      | はい     | フォーム送信時に反映されるチェック状態                             |
| `text`      | `text`      | `string`  | `"checkbox"` | いいえ   | トグルの横に表示されるラベル文字列                                 |
| `name`      | `name`      | `string`  | `""`         | いいえ   | フォーム送信時に使用される入力フィールド名                         |
| `value`     | `value`     | `string`  | `"on"`       | いいえ   | チェック時に送信されるフォームペイロード値                         |
| `placement` | `placement` | `string`  | `"after"`    | いいえ   | ラベルに対するトグルの配置位置（`"after"`または`"before"`）        |
| `disabled`  | `disabled`  | `boolean` | `false`      | いいえ   | 操作を無効化しフォーム送信ペイロードから除外する                   |

## プログラム用api

```javascript
const checkbox = select('d-checkbox');

checkbox.checked = true;
checkbox.text = 'allow telemetry';
checkbox.placement = 'before';
checkbox.disabled = false;
checkbox.name = 'telemetry';
checkbox.value = 'accepted';
```

## イベント

| イベント | 詳細                    | バブリング | 説明                                             |
| :------- | :---------------------- | :--------- | :----------------------------------------------- |
| `change` | boolean（チェック状態） | はい       | ユーザーがチェックボックスを切り替えたときに送出 |

## 動作仕様

- ユーザーによる切り替え時またはjavascriptによる変更時にチェック状態、ラベル文字列、フォーム送信値を自動的に同期します。
