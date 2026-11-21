# d-datepicker

`<d-popover>`と`<d-calendar>`を組み合わせ、ネイティブスタイルのトリガー、クリアボタン、単一・範囲選択モード、日付のローカライズ表示、キーボードショートカットを備えたフォーム連動の日付選択コンポーネントです。

```html
<!-- プレースホルダー付き単一選択 -->
<d-datepicker name="due_date" placeholder="pick a date"></d-datepicker>

<!-- 初期値とローカライズ表示、クリアボタン有効 -->
<d-datepicker name="event_date" value="2026-11-19" format="localized" clearable></d-datepicker>

<!-- 範囲選択 -->
<d-datepicker name="reservation" mode="range" value="2026-11-10,2026-11-20" clearable></d-datepicker>

<!-- 選択可能範囲の制限 -->
<d-datepicker min="2026-11-01" max="2026-11-30"></d-datepicker>
```

<a name="属性とプロパティd-datepicker"></a>
## 属性とプロパティ（d-datepicker）

| 属性                | プロパティ        | 型        | デフォルト値   | フォーム | 説明                                                                          |
| :------------------ | :---------------- | :-------- | :------------- | :------- | :---------------------------------------------------------------------------- |
| `name`              | `name`            | `string`  | `""`           | いいえ   | FormDataやネイティブフォーム送信用のフィールド名                              |
| `value`             | `value`           | `string`  | `""`           | はい     | 選択値（`YYYY-MM-DD`または範囲`from,to`）                                     |
| `mode`              | `mode`            | `string`  | `"single"`     | いいえ   | 選択モード（`"single"`、`"multiple"`、または`"range"`）                       |
| `placeholder`       | `placeholder`     | `string`  | `""`           | いいえ   | 未選択時のプレースホルダー（未指定時は`"pick a date"`または`"pick date range"`）|
| `format`            | `format`          | `string`  | `""`           | いいえ   | 表示フォーマット（`"localized"`で地域言語形式、空文字でISO形式）             |
| `era`               | `era`             | `string`  | `""`           | いいえ   | カレンダーの年号表示形式（和暦表記の場合は`"japanese"`）                      |
| `min`               | `min`             | `string`  | `""`           | いいえ   | 選択可能な最小日付（`YYYY-MM-DD`）                                            |
| `max`               | `max`             | `string`  | `""`           | いいえ   | 選択可能な最大日付（`YYYY-MM-DD`）                                            |
| `locale`            | `locale`          | `string`  | `""`           | いいえ   | カレンダーおよび日付表示に使用するBCP 47ロケールコード                        |
| `clearable`         | `clearable`       | `boolean` | `false`        | いいえ   | 日付選択時にクリアボタンを表示するかどうか                                    |
| `disabled`          | `disabled`        | `boolean` | `false`        | いいえ   | トリガーおよび内部カレンダーの操作を無効化                                    |
| `week-starts-on`    | `weekStartsOn`    | `number`  | `0`            | いいえ   | カレンダーの開始曜日（`0` 日曜日、`1` 月曜日）                                |
| `show-outside-days` | `showOutsideDays` | `boolean` | `true`         | いいえ   | カレンダーに前後の月の日付を表示するかどうか                                  |
| `close-on-select`   | `closeOnSelect`   | `boolean` | `true`         | いいえ   | 日付または範囲の選択完了後にポップオーバーを自動で閉じるかどうか              |
| `side`              | `side`            | `string`  | `"bottom"`     | いいえ   | ポップオーバーの配置方向（`"top"`または`"bottom"`）                           |
| `align`             | `align`           | `string`  | `"center"`     | いいえ   | ポップオーバーの揃え位置（`"start"`、`"center"`、`"end"`）                    |

<a name="プログラム用apid-datepicker"></a>
## プログラム用api（d-datepicker）

```javascript
const picker = select('d-datepicker');

// ポップオーバーの操作
picker.open();
picker.close();
picker.toggle();
picker.clear();

// 状態ゲッター
picker.isOpen;        // boolean
picker.selectedDate;  // 単一選択時のDateオブジェクトまたはnull
picker.range;         // 範囲選択時の{ from: Date, to: Date }
picker.value;         // string
```

<a name="イベントd-datepicker"></a>
## イベント（d-datepicker）

| イベント | ペイロード                                     | バブリング | 説明                                 |
| :------- | :--------------------------------------------- | :--------- | :----------------------------------- |
| `change` | `{ value, from, to, date, dates }`             | はい       | 日付の選択状態が変更されたとき       |
| `input`  | `{ value }`                                    | はい       | 値が更新されたとき                   |
| `open`   | `{}`                                           | はい       | カレンダーポップオーバーが開いたとき |
| `close`  | `{}`                                           | はい       | カレンダーポップオーバーが閉じたとき |

<a name="動作仕様d-datepicker"></a>
## 動作仕様（d-datepicker）

- カレンダーアイコン、日付プレビューまたはプレースホルダー、クリアボタンを備えたネイティブライクなトリガーを描画します。
- `<d-popover>`内部に`<d-calendar>`をフロート配置し、画面端との衝突検知を行いながらアンカーに追従します。
- 単一選択モードでは、日付をクリックするとトリガー表示が更新され、滑らかなスプリングアニメーションでポップオーバーが自動で閉じます。
- 範囲選択モードでは、`YYYY-MM-DD - YYYY-MM-DD`形式で表示され、開始日と終了日の双方が選択された時点で自動で閉じます。
- フルキーボード操作に対応（`Enter`、`Space`、`ArrowDown`で展開、`Escape`で閉じる、クリア可能時は`Backspace`または`Delete`で日付をクリア）。
- フォーム連動（`static form = true`）に対応し、通常のフォーム送信や`FormData`に自然に参加します。
