# d-image-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/image-input.mp4" type="video/mp4">
  </video>
</p>

即時クライアントプレビューとアクションオーバーレイを備えたドラッグアンドドロップ画像ドロップゾーンです。

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="cover" placeholder="drop profile photo"></d-image-input>
```

## 属性とプロパティ

| 属性           | プロパティ    | 型                | デフォルト値                         | フォーム | 説明                                                   |
| :------------- | :------------ | :---------------- | :----------------------------------- | :------- | :----------------------------------------------------- |
| `accept`       | `accept`      | `string`          | `"image/png, image/jpeg, image/gif"` | いいえ   | 許可する画像MIMEタイプ                                 |
| `icon`         | `icon`        | `string`          | `"dstrn-pic-line"`                   | いいえ   | プレースホルダーアイコンクラス                         |
| `placeholder`  | `placeholder` | `string`          | `"drag & drop image"`                | いいえ   | ドロップゾーンのタイトル文字列                         |
| `subtitle`     | `subtitle`    | `string`          | `"or click to browse"`               | いいえ   | ドロップゾーンのサブタイトル文字列                     |
| `replace-text` | `replaceText` | `string`          | `"replace"`                          | いいえ   | オーバーレイの置換ボタン文字列                         |
| `delete-text`  | `deleteText`  | `string`          | `"delete"`                           | いいえ   | オーバーレイの削除ボタン文字列                         |
| `no-replace`   | `noReplace`   | `boolean`         | `false`                              | いいえ   | trueの場合に置換ボタンを非表示化                       |
| `no-delete`    | `noDelete`    | `boolean`         | `false`                              | いいえ   | trueの場合に削除ボタンを非表示化                       |
| `fit`          | `fit`         | `string`          | `"contain"`                          | いいえ   | object-fit CSSプロパティ（`contain`、`cover`、`fill`） |
| `name`         | `name`        | `string`          | `""`                                 | いいえ   | フォームペイロードフィールド名                         |
| `id`           | `id`          | `string`          | `""`                                 | いいえ   | コンポーネントID                                       |
| `value`        | `value`       | `object`/`string` | `null`                               | はい     | `File`オブジェクトまたは画像URL文字列                  |

## プログラム用api

```javascript
const imgInput = select('d-image-input');

imgInput.reset();
imgInput.preview = '/img/avatar.png';

console.log(imgInput.value);
```

## イベント

| イベント | 詳細               | バブリング | 説明                                             |
| :------- | :----------------- | :--------- | :----------------------------------------------- |
| `change` | `File`または`null` | はい       | 画像が選択、ドロップ、または削除されたときに送出 |

## 動作仕様

- 即時プレビューと画像の置換や削除を行うアクションボタンを備えたドラッグアンドドロップ画像選択を提供します。
- フォーム送信と自動統合されアップロード時にエラー状態をクリアします。
