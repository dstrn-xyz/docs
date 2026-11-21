# d-skeleton

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/skeleton.mp4" type="video/mp4">
  </video>
</p>

カスタマイズ可能な寸法、行数、カードプリセットを備えたコンテンツ読み込み用プレースホルダーシェイプです。

```html
<d-skeleton type="text" lines="3" gap="0.5em"></d-skeleton>
<d-skeleton type="circle" size="3em"></d-skeleton>
<d-skeleton type="rect" width="100%" height="150px" radius="0.5em"></d-skeleton>
<d-skeleton type="card"></d-skeleton>
```

## 属性とプロパティ

| 属性     | プロパティ | 型       | デフォルト値 | 説明                                                           |
| :------- | :--------- | :------- | :----------- | :------------------------------------------------------------- |
| `type`   | `type`     | `string` | `"text"`     | プリセットシェイプ（`"text"`、`"circle"`、`"rect"`、`"card"`） |
| `lines`  | `lines`    | `number` | `1`          | `"text"`タイプの行数（最終行は60%の幅でレンダリング）          |
| `width`  | `width`    | `string` | `null`       | CSS幅                                                          |
| `height` | `height`   | `string` | `null`       | CSS高さ                                                        |
| `size`   | `size`     | `string` | `null`       | circleおよびrectでの等しい幅と高さのショートハンド             |
| `gap`    | `gap`      | `string` | `"0.6em"`    | テキストモードでの行間の垂直間隔                               |
| `radius` | `radius`   | `string` | `null`       | カスタムborder-radiusの上書き                                  |
