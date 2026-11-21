# d-file-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/file-input.mp4" type="video/mp4">
  </video>
</p>

ファイルリスト表示、バイトサイズ計算、コンパクトバーモードを備えたドラッグアンドドロップファイルアップロードコンテナーです。

```html
<d-file-input accept=".pdf,.zip,.csv" name="documents" multiple compact></d-file-input>
```

## 属性とプロパティ

| 属性          | プロパティ    | 型               | デフォルト値           | フォーム | 説明                                           |
| :------------ | :------------ | :--------------- | :--------------------- | :------- | :--------------------------------------------- |
| `accept`      | `accept`      | `string`         | `"*/*"`                | いいえ   | 許可するMIMEタイプまたはファイル拡張子         |
| `icon`        | `icon`        | `string`         | `"dstrn-folder-line"`  | いいえ   | ドロップゾーンに表示されるアイコンクラス       |
| `placeholder` | `placeholder` | `string`         | `"drag & drop file"`   | いいえ   | プライマリドロップゾーンタイトル               |
| `subtitle`    | `subtitle`    | `string`         | `"or click to browse"` | いいえ   | セカンダリヘルパーサブタイトル                 |
| `compact`     | `compact`     | `boolean`        | `true`                 | いいえ   | trueの場合にスリムな水平バーとしてレンダリング |
| `multiple`    | `multiple`    | `boolean`        | `false`                | いいえ   | 複数ファイルの選択およびドロップを許可         |
| `name`        | `name`        | `string`         | `""`                   | いいえ   | フォームペイロードフィールド名                 |
| `id`          | `id`          | `string`         | `""`                   | いいえ   | コンポーネントID                               |
| `value`       | `value`       | `object`/`array` | `null`                 | はい     | `File`、`File[]`、またはURL文字列ペイロード    |

## プログラム用api

```javascript
const fileInput = select('d-file-input');

fileInput.reset();

console.log(fileInput.value);
```

## イベント

| イベント | 詳細                           | バブリング | 説明                                                 |
| :------- | :----------------------------- | :--------- | :--------------------------------------------------- |
| `change` | `File`、`File[]`、または`null` | はい       | ファイルが選択、ドロップ、または削除されたときに送出 |

## 動作仕様

- ファイルリスト表示、フォーマットされたファイルサイズ（Bytes、KB、MB、GB）、削除ボタンを備えたドラッグアンドドロップファイルアップロードを提供します。
- 有効なファイルがドロップされたときにエラー状態を自動的にクリアします。
- 標準フォーム送信と統合されます。
