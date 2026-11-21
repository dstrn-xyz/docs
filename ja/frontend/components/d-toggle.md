# d-toggle

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/toggle.mp4" type="video/mp4">
  </video>
</p>

マグネティックホバーインジケーターとアニメーション遷移を備えたセグメント化トグルコントロールです。

```html
<d-toggle name="billing_period" value="0">
  <div default>monthly billing</div>
  <div>annual billing (save 20%)</div>
</d-toggle>
```

## 属性とプロパティ

| 属性              | プロパティ        | 型       | デフォルト値                  | フォーム | 説明                                              |
| :---------------- | :---------------- | :------- | :---------------------------- | :------- | :------------------------------------------------ |
| `value`           | `value`           | `number` | `0`                           | はい     | 現在選択されている項目の0始まりの整数インデックス |
| `name`            | `name`            | `string` | `""`                          | いいえ   | フォームフィールド名                              |
| `id`              | `id`              | `string` | `""`                          | いいえ   | コンポーネントID                                  |
| `color`           | `color`           | `string` | `"var(--container-l)"`        | いいえ   | コンテナー背景カラー変数                          |
| `activecolor`     | `activeColor`     | `string` | `"var(--accent)"`             | いいえ   | アクティブ項目のハイライト色                      |
| `textcolor`       | `textColor`       | `string` | `""`                          | いいえ   | 非アクティブ項目のテキスト色（デフォルトは自動コントラスト） |
| `textactivecolor` | `textActiveColor` | `string` | `""`                          | いいえ   | アクティブ項目のテキスト色（デフォルトは自動コントラスト） |
| `bgcolor`         | `bgColor`         | `string` | `"var(--container-l)"`        | いいえ   | ベース背景色                                      |
| `hovercolor`      | `hoverColor`      | `string` | `""`                          | いいえ   | ホバー背景色（デフォルトはテーマ適応）            |

## プログラム用api

```javascript
const toggle = select('d-toggle');

console.log(toggle.selected);
toggle.selected = 1;
toggle.value = 0;
```

## イベント

| イベント | 詳細                 | バブリング | 説明                                   |
| :------- | :------------------- | :--------- | :------------------------------------- |
| `change` | 選択インデックス数値 | はい       | アクティブな選択が変更されたときに送出 |

## 動作仕様

- ホバーおよび選択のアニメーション遷移を伴うインタラクティブなセグメント選択肢をレンダリングします。
- `default`属性または数値による初期選択をサポートします。
