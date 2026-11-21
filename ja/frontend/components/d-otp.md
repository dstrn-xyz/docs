# d-otp

単一のネイティブ入力でSMS自動入力、ペースト時の自動サニタイズ、キーボード編集、分割スロットグループを実現するフォーム関連のワンタイムパスワード入力コンポーネントです。

```html
<d-otp length="6" separator="3" name="code"></d-otp>
<d-otp length="4" mask name="pin"></d-otp>
<d-otp length="6" pattern="alphanumeric" uppercase separator="2"></d-otp>
```

<a name="属性とプロパティd-otp"></a>
## 属性とプロパティ（d-otp）

| 属性        | プロパティ  | 型        | デフォルト値 | フォーム | 説明                                                                     |
| :---------- | :---------- | :-------- | :----------- | :------- | :----------------------------------------------------------------------- |
| `length`    | `length`    | `number`  | `6`          | いいえ   | スロット総数（1から16）                                                  |
| `value`     | `value`     | `string`  | `""`         | はい     | フォーム送信時に反映されるサニタイズ済みOTP文字列                        |
| `name`      | `name`      | `string`  | `""`         | いいえ   | フォーム送信時のフィールド名                                             |
| `pattern`   | `pattern`   | `string`  | `"digits"`   | いいえ   | 許可文字プリセット（`"digits"`、`"alphanumeric"`、`"alpha"`）または正規表現 |
| `separator` | `separator` | `string`  | `""`         | いいえ   | グループ区切り間隔（`"3"`等）またはカンマ区切りの分割位置（`"2,4"`等）   |
| `mask`      | `mask`      | `boolean` | `false`      | いいえ   | PIN入力向けに文字をドット表示にマスクする                                |
| `disabled`  | `disabled`  | `boolean` | `false`      | いいえ   | 入力を無効化しフォーム送信から除外する                                   |
| `invalid`   | `invalid`   | `boolean` | `false`      | いいえ   | 全スロットにエラーボーダー状態を適用                                     |
| `autofocus` | `autofocus` | `boolean` | `false`      | いいえ   | マウント時に自動フォーカスする                                           |
| `uppercase` | `uppercase` | `boolean` | `false`      | いいえ   | 入力英字を大文字に変換して検証する                                       |

<a name="プログラム用apid-otp"></a>
## プログラム用api（d-otp）

```javascript
const otp = select('d-otp');

otp.focus();
otp.error('invalid verification code');
otp.clearError();
otp.clear();
```

<a name="イベントd-otp"></a>
## イベント（d-otp）

| イベント   | 詳細             | バブリング | 説明                                         |
| :--------- | :--------------- | :--------- | :------------------------------------------- |
| `input`    | string（`value`） | はい       | サニタイズ済みOTP値が変更されたときに送出     |
| `change`   | string（`value`） | はい       | OTP値が更新されたときに送出                  |
| `complete` | string（`value`） | はい       | 全スロットが入力完了したときに一度送出       |

<a name="動作仕様d-otp"></a>
## 動作仕様（d-otp）

- 単一の`<input autocomplete="one-time-code">`を背面に配置しているため、モバイル自動入力や区切り文字付きコードの貼り付け、バックスペース削除が自然に動作します。
- 入力文字のポップアニメーション、空アクティブスロットの点滅キャレット、グローを使わないクリアなボーダー色変化、`error()`によるシェイク表示を備え、再入力時にエラー状態を自動解除します。
