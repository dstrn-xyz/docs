# d-calendar

単一・範囲・複数選択に対応し、ネイティブ`Intl.DateTimeFormat`によるローカライズ表記、滑らかな月カルーセルスライドアニメーション、フルキーボード操作を備えたゼロ依存のインタラクティブカレンダーです。

```html
<!-- 単一選択 -->
<d-calendar mode="single" value="2026-11-19"></d-calendar>

<!-- 範囲選択（開始・中間・終了の連続ハイライト表示） -->
<d-calendar mode="range" value="2026-11-10,2026-11-20"></d-calendar>

<!-- 複数日選択 -->
<d-calendar mode="multiple" value="2026-11-05,2026-11-12,2026-11-19"></d-calendar>

<!-- 選択可能範囲の制限 -->
<d-calendar min="2026-11-01" max="2026-11-30"></d-calendar>
```

<a name="属性とプロパティd-calendar"></a>
## 属性とプロパティ（d-calendar）

| 属性                | プロパティ        | 型        | デフォルト値 | フォーム | 説明                                                          |
| :------------------ | :---------------- | :-------- | :----------- | :------- | :------------------------------------------------------------ |
| `mode`              | `mode`            | `string`  | `"single"`   | いいえ   | 選択モード（`"single"`、`"range"`、`"multiple"`）             |
| `value`             | `value`           | `string`  | `""`         | いいえ   | 選択値（`YYYY-MM-DD`、範囲`from,to`、またはカンマ区切り一覧） |
| `month`             | `month`           | `number`  | `null`       | いいえ   | 表示月インデックス（`1`〜`12`）                               |
| `year`              | `year`            | `number`  | `null`       | いいえ   | 表示年（例：`2026`）                                          |
| `min`               | `min`             | `string`  | `""`         | いいえ   | 選択可能な最小日付（`YYYY-MM-DD`）                            |
| `max`               | `max`             | `string`  | `""`         | いいえ   | 選択可能な最大日付（`YYYY-MM-DD`）                            |
| `locale`            | `locale`          | `string`  | `""`         | いいえ   | BCP 47ロケールコード（デフォルトは文書またはブラウザの言語）  |
| `era`               | `era`             | `string`  | `""`         | いいえ   | カレンダーの年号表示形式（和暦表記の場合は`"japanese"`）      |
| `week-starts-on`    | `weekStartsOn`    | `number`  | `0`          | いいえ   | 週の開始曜日（`0` 日曜日、`1` 月曜日）                        |
| `show-outside-days` | `showOutsideDays` | `boolean` | `true`       | いいえ   | 前後の月の日付を表示するかどうか                              |
| `disabled`          | `disabled`        | `boolean` | `false`      | いいえ   | 操作を無効化しグリッドを半透明表示                            |

<a name="プログラム用apid-calendar"></a>
## プログラム用api（d-calendar）

```javascript
const cal = select('d-calendar');

// 月の移動
cal.prevMonth();
cal.nextMonth();

// ゲッター
cal.viewMode;      // "days", "months", または "years"
cal.selectedDate;  // 単一選択時のDateオブジェクトまたはnull
cal.range;         // 範囲選択時の{ from: Date, to: Date }
cal.selectedDates; // 複数選択時のDate配列
```

<a name="イベントd-calendar"></a>
## イベント（d-calendar）

| イベント       | ペイロード                            | バブリング | 説明                               |
| :------------- | :------------------------------------ | :--------- | :--------------------------------- |
| `select`       | `{ value, date }`                     | はい       | いずれかの日付がクリックされたとき |
| `change`       | `{ value, date }`（単一）、`{ value, from, to }`（範囲）、または`{ value, dates }`（複数） | はい       | 選択状態が更新されたとき           |
| `month-change` | `{ year, month, monthString, value }` | はい       | 新しい月にナビゲーションされたとき |

<a name="動作仕様d-calendar"></a>
## 動作仕様（d-calendar）

- 月の切り替え時は移動方向に応じた滑らかなカルーセルスライドアニメーション（`translateX` + 不透明度）が再生されます。
- ヘッダーの月または年テキストをクリックすると、12個の選択グリッド（月選択または10年単位の年選択）へズーム展開して素早く移動できます。
- 範囲選択モードでは、開始日と終了日をドラッグまたはクリックすることで、開始・中間・終了が連結したシームレスなトラックとして描画されます。
- 曜日はIntl.DateTimeFormatに基づきローカライズされたコンパクトな2文字表記で均等配置されます。
- フルキーボード操作に対応（`ArrowLeft` / `ArrowRight`で1日移動、`ArrowUp` / `ArrowDown`で1週間移動、`PageUp` / `PageDown`で月移動、`Enter` / `Space`で選択）。
- `min`および`max`の境界外の日付は自動的に非活性化され、選択不可となります。
