# d-radiobutton

同一グループ間の自動排他制御、オプションの説明文、配置切り替え、キーボード矢印移動を備えたフォーム関連のラジオボタンです。

```html
<d-radiobutton name="plan" value="free" text="starter" description="for personal projects" checked></d-radiobutton>
<d-radiobutton name="plan" value="pro" text="pro" description="unlimited workspaces"></d-radiobutton>
<d-radiobutton name="plan" value="team" text="team" placement="after"></d-radiobutton>
```

<a name="属性とプロパティd-radiobutton"></a>
## 属性とプロパティ（d-radiobutton）

| 属性          | プロパティ    | 型        | デフォルト値 | フォーム | 説明                                                               |
| :------------ | :------------ | :-------- | :----------- | :------- | :----------------------------------------------------------------- |
| `checked`     | `checked`     | `boolean` | `false`      | はい     | 選択状態（選択すると同じ`name`の他ラジオボタンが自動解除）         |
| `name`        | `name`        | `string`  | `""`         | いいえ   | グループおよびフォーム送信時のフィールド名                         |
| `value`       | `value`       | `string`  | `"on"`       | いいえ   | 選択時にフォーム送信される値                                       |
| `text`        | `text`        | `string`  | `""`         | いいえ   | ラベル文字列（未指定時は内部テキストを使用）                       |
| `description` | `description` | `string`  | `""`         | いいえ   | ラベル下部に表示する補足説明テキスト                               |
| `placement`   | `placement`   | `string`  | `"before"`   | いいえ   | ラベルに対するラジオ円の配置（`"before"`または`"after"`）          |
| `disabled`    | `disabled`    | `boolean` | `false`      | いいえ   | 操作を無効化しフォーム送信から除外する                             |

<a name="プログラム用apid-radiobutton"></a>
## プログラム用api（d-radiobutton）

```javascript
const proRadio = select('d-radiobutton[value="pro"]');
proRadio.checked = true;
proRadio.description = 'priority support included';
```

<a name="イベントd-radiobutton"></a>
## イベント（d-radiobutton）

| イベント | 詳細             | バブリング | 説明                                         |
| :------- | :--------------- | :--------- | :------------------------------------------- |
| `change` | string（`value`） | はい       | ユーザーがこのラジオボタンを選択したときに送出 |

<a name="動作仕様d-radiobutton"></a>
## 動作仕様（d-radiobutton）

- 同じ`<form>`、`<d-form>`、またはドキュメント内で同じ`name`を共有する`<d-radiobutton>`同士を自動的に排他同期します。
- `ArrowUp`/`ArrowLeft`、`ArrowDown`/`ArrowRight`、`Space`、`Enter`によるキーボード操作に対応し、無効項目をスキップします。
- `d-checkbox`と共通のスプリングアニメーション曲線を使用し、`prefers-reduced-motion`にも対応します。
