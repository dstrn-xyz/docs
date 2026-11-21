# d-progress

オプションのラベル、パーセンテージ表示、任意の範囲設定、不確定ローディングアニメーションを備えたGPUコンポジット型プログレスバーです。

```html
<d-progress value="33"></d-progress>
<d-progress value="56" label="upload progress" show-value></d-progress>
<d-progress indeterminate label="syncing assets"></d-progress>
```

<a name="属性とプロパティd-progress"></a>
## 属性とプロパティ（d-progress）

| 属性            | プロパティ      | 型        | デフォルト値 | フォーム | 説明                                                       |
| :-------------- | :-------------- | :-------- | :----------- | :------- | :--------------------------------------------------------- |
| `value`         | `value`         | `number`  | `0`          | いいえ   | `min`と`max`の範囲内における現在の進捗値                   |
| `min`           | `min`           | `number`  | `0`          | いいえ   | 割合計算の最小値                                           |
| `max`           | `max`           | `number`  | `100`        | いいえ   | 100パーセント完了を表す最大値                              |
| `indeterminate` | `indeterminate` | `boolean` | `false`      | いいえ   | 連続スライドの不確定ローディングモードに切り替え           |
| `label`         | `label`         | `string`  | `""`         | いいえ   | トラック上部に表示するオプションのラベル                   |
| `show-value`    | `showValue`     | `boolean` | `false`      | いいえ   | トラック上部にパーセンテージ表示を追加                     |
| `color`         | `color`         | `string`  | `""`         | いいえ   | バー色のカスタムCSS値（デフォルトは`var(--accent)`）       |
| `size`          | `size`          | `string`  | `""`         | いいえ   | トラック高さのカスタムCSS値（デフォルトは`0.25em`）        |

<a name="プログラム用apid-progress"></a>
## プログラム用api（d-progress）

```javascript
const progress = select('d-progress');

progress.value = 72;
console.log(progress.percentage);
console.log(progress.ratio);
```

<a name="イベントd-progress"></a>
## イベント（d-progress）

| イベント   | 詳細             | バブリング | 説明                                               |
| :--------- | :--------------- | :--------- | :------------------------------------------------- |
| `complete` | number（`value`） | はい       | 確定モードで`value`が`max`に到達したときに一度送出 |

<a name="動作仕様d-progress"></a>
## 動作仕様（d-progress）

- `transform: scaleX()`のみでアニメーションするためレイアウト再計算を発生させず、`d-slider`などの高頻度更新でも滑らかに動作します。
- 画面外に出た際は不確定アニメーションを自動一時停止し、`prefers-reduced-motion`にも対応します。
- 完了状態に達した際に一度だけ控えめな光沢アニメーションを再生します。
