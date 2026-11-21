# d-icon-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/icon-button.mp4" type="video/mp4">
  </video>
</p>

カスタムアイコン、サイズ、動的テーマアクティブカラーをサポートするアイコンボタンラッパーです。

```html
<d-icon-button icon="dstrn-heart-line" size="2em" color="var(--content-l)" activecolor="var(--red)"></d-icon-button>
```

## 属性とプロパティ

| 属性          | プロパティ    | 型       | デフォルト値         | フォーム | 説明                                                        |
| :------------ | :------------ | :------- | :------------------- | :------- | :---------------------------------------------------------- |
| `icon`        | `icon`        | `string` | `""`                 | いいえ   | CSSアイコンクラス（例: `dstrn-heart-line`、`dstrn-search`） |
| `size`        | `size`        | `string` | `"4.2em"`            | いいえ   | アイコンコンテナーのフォントサイズ                          |
| `color`       | `color`       | `string` | `"var(--content-l)"` | いいえ   | デフォルトのアイコン色                                      |
| `activecolor` | `activeColor` | `string` | `"var(--accent)"`    | いいえ   | アクティブ時のアイコン色                                    |
| `id`          | `id`          | `string` | `""`                 | いいえ   | コンポーネントID                                            |
