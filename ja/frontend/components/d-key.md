# d-key

キーボードのキーキャップバッジおよびフォーム関連のショートカット記録入力コンポーネントです。ショートカット文字列をキーキャップに自動分割し、プラットフォームに応じた記号表示、物理キー押下のライブ反映、クリックによるショートカット入力に対応しています。

```html
<d-key>cmd+k</d-key>
<d-key separator="">cmd,shift,p</d-key>
<d-key live>cmd+s</d-key>
<d-key editable name="shortcut" value="cmd+b"></d-key>

<span class="d-key-group">
  <d-key>cmd+b</d-key>
  <d-key>cmd+k</d-key>
</span>
```

<a name="属性とプロパティd-key"></a>
## 属性とプロパティ（d-key）

| 属性          | プロパティ    | 型        | デフォルト値  | フォーム | 説明                                                               |
| :------------ | :------------ | :-------- | :------------ | :------- | :----------------------------------------------------------------- |
| `value`       | `value`       | `string`  | `""`          | はい     | 正規化されたショートカット文字列（テキスト指定時も`.value`で取得可） |
| `name`        | `name`        | `string`  | `""`          | いいえ   | フォーム送信時に使用される入力フィールド名                         |
| `separator`   | `separator`   | `string`  | `"+"`         | いいえ   | キーキャップ間の区切り文字（`""`で区切りなし）                     |
| `editable`    | `editable`    | `boolean` | `false`       | いいえ   | クリックでショートカットを記録する入力モードを有効化               |
| `live`        | `live`        | `boolean` | `false`       | いいえ   | 対応する物理キー押下時にキーキャップをアニメーション表示           |
| `placeholder` | `placeholder` | `string`  | `"press key"` | いいえ   | 編集可能モードで値が空のときに表示するプレースホルダー             |
| `disabled`    | `disabled`    | `boolean` | `false`       | いいえ   | 入力を無効化しフォーム送信から除外する                             |
| `clearable`   | `clearable`   | `boolean` | `true`        | いいえ   | 記録中に`Backspace`または`Delete`でショートカットをクリア可能にする |

<a name="プログラム用apid-key"></a>
## プログラム用api（d-key）

```javascript
const key = select('d-key');

console.log(key.value);
console.log(key.keys);

key.value = 'ctrl+shift+p';
key.clear();
```

<a name="イベントd-key"></a>
## イベント（d-key）

| イベント | 詳細                           | バブリング | 説明                                                 |
| :------- | :----------------------------- | :--------- | :--------------------------------------------------- |
| `change` | string（正規化されたキー配列） | はい       | 入力モードでショートカットが記録またはクリアされたときに送出 |

<a name="動作仕様d-key"></a>
## 動作仕様（d-key）

- `value`属性または初期テキスト内の`+`や`,`区切りショートカットを解析し、常に`.value`で正規化文字列、`.keys`でトークン配列を取得できます。
- プラットフォームに応じて修飾キー記号を自動変換します（macでは`cmd`および`mod`を`⌘`、`alt`を`⌥`、その他では`ctrl`/`alt`として表示、`shift`は`⇧`、`enter`は`⏎`）。
- `editable`モードではクリックまたはフォーカスで記録を開始し、修飾キー押下中はリアルタイムにプレビュー、通常キー入力で確定して`change`を送出、`Escape`でキャンセル、`Backspace`/`Delete`でクリアします。
- `.btn`内での表示や`.d-key-group`でのグループ化に対応しています。
