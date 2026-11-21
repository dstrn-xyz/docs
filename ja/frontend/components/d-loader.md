# d-loader

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/loader.mp4" type="video/mp4">
  </video>
</p>

ビューポート交差による自動一時停止機能を備えた高性能キャンバス円弧スピナーです。

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

## 属性とプロパティ

| 属性    | プロパティ | 型       | デフォルト値 | フォーム | 説明                                                       |
| :------ | :--------- | :------- | :----------- | :------- | :--------------------------------------------------------- |
| `color` | `color`    | `string` | `""`         | いいえ   | 線の色（デフォルトは`--accent`または計算されたテキスト色） |

## プログラム用api

```javascript
const loader = select('d-loader');

loader.destroy();
```

## 動作仕様

- CPU使用を節約するため画面外にスクロールされたときにアニメーションを自動一時停止する軽量キャンバススピナーです。
- コンテナーの寸法に自動適応し高解像度画面でも鮮明にレンダリングされます。
