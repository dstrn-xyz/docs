# d-slider

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/slider.mp4" type="video/mp4">
  </video>
</p>

ポインタードラッグ、キーボードナビゲーション、音声フィードバック、自動インジケーターテキストバインディングを備えたレンジスライダーです。

```html
<d-slider id="volume" name="volume" min="0" max="1" step="0.01" value="0.75"></d-slider>
<span data-slider-volume class="text-content-l">75%</span>
```

## 属性とプロパティ

| 属性         | プロパティ   | 型        | デフォルト値            | フォーム | 説明                                                               |
| :----------- | :----------- | :-------- | :---------------------- | :------- | :----------------------------------------------------------------- |
| `value`      | `value`      | `number`  | `0.5`                   | はい     | 現在の数値スライダー値                                             |
| `min`        | `min`        | `number`  | `0`                     | いいえ   | 許容される最小値                                                   |
| `max`        | `max`        | `number`  | `1`                     | いいえ   | 許容される最大値                                                   |
| `step`       | `step`       | `number`  | `0.01`                  | いいえ   | ドラッグおよび矢印キーのステップ刻み値                             |
| `name`       | `name`       | `string`  | `""`                    | いいえ   | フォームペイロードフィールド名                                     |
| `id`         | `id`         | `string`  | `""`                    | いいえ   | コンポーネントID（`data-slider-{id}`テキストバインディングに使用） |
| `disabled`   | `disabled`   | `boolean` | `false`                 | いいえ   | スライダー操作を無効化                                             |
| `thumbcolor` | `thumbColor` | `string`  | `"var(--accent)"`       | いいえ   | つまみのカラー変数                                                 |
| `fillcolor`  | `fillColor`  | `string`  | `"var(--accent-d)"`     | いいえ   | アクティブトラックの塗りつぶしカラー変数                           |
| `trackcolor` | `trackColor` | `string`  | `"var(--border-muted)"` | いいえ   | 非アクティブトラックの背景カラー変数                               |
| `sfx`        | `sfx`        | `string`  | 組み込み                | いいえ   | 効果音のオーディオURLまたはbase64オーディオペイロード              |

## プログラム用api

```javascript
const slider = select('d-slider');

const current = slider.getValue();
slider.setValue(0.85);
```

## イベント

| イベント | 詳細 | バブリング | 説明                                         |
| :------- | :--- | :--------- | :------------------------------------------- |
| `input`  | 数値 | はい       | 連続ドラッグおよび矢印キー移動中に送出       |
| `change` | 数値 | はい       | ポインターアップまたはキーボード確定時に送出 |

## 動作仕様

- 滑らかなドラッグとキーボード矢印ナビゲーションをサポートします。
- `data-slider-{id}`を持つ要素のテキストを現在のパーセンテージ値で自動更新します。
- フォーム送信と統合されます。
