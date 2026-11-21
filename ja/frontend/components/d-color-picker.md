# d-color-picker

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/color-picker.mp4" type="video/mp4">
  </video>
</p>

2D彩度および明度コントロール、色相スライダー、アルファ透明度コントロールを備えた高精度カラーピッカーです。

```html
<d-color-picker value="#3b82f6" name="theme_color" label="accent color"></d-color-picker>
```

## 属性とプロパティ

| 属性     | プロパティ | 型        | デフォルト値     | フォーム | 説明                                                         |
| :------- | :--------- | :-------- | :--------------- | :------- | :----------------------------------------------------------- |
| `value`  | `value`    | `string`  | `"#d3ac5f"`      | はい     | 現在の16進数カラー値（3桁、4桁、6桁、8桁の16進数をサポート） |
| `name`   | `name`     | `string`  | `""`             | いいえ   | フォーム送信用の入力名属性                                   |
| `id`     | `id`       | `string`  | `""`             | いいえ   | 要素識別子                                                   |
| `label`  | `label`    | `string`  | `"color picker"` | いいえ   | ピッカーの上部に表示されるヘッダーラベル                     |
| `opened` | `opened`   | `boolean` | `false`          | いいえ   | ポップアップピッカーパネルが初期状態で開いているかどうか     |

## プログラム用api

```javascript
const picker = select('d-color-picker');

picker.open();
picker.close();
picker.toggle();
picker.setHex('#ef4444');

console.log(picker.hex);
console.log(picker.h, picker.s, picker.b, picker.a);
```

## イベント

| イベント | 詳細         | バブリング | 説明                                                       |
| :------- | :----------- | :--------- | :--------------------------------------------------------- |
| `change` | 16進数文字列 | はい       | ドラッグ、入力、またはsetHexによって色が変化したときに送出 |
| `input`  | 16進数文字列 | はい       | ドラッグ操作中に連続して送出                               |

## 動作仕様

- 市松模様の透明度背景を持つインタラクティブな2Dカラープレーン、色相バー、アルファスライダーを提供します。
- 内部`d-popover`トップレイヤーでパネルを表示するため、`overflow: hidden`によるクリッピングを受けず、画面端での自動反転、スクロール追従、外部クリックや`Escape`での自動クローズに対応します。
- コンポーネントの境界外でも滑らかなドラッグ操作をサポートします。
- 要素のリサイズ時に自動的に再描画され高解像度ディスプレイでも鮮明にレンダリングされます。
- リセットアイコンをクリックすると初期カラー値に戻ります。
