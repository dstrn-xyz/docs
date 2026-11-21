# d-codeblock

アニメーション付きコピーボタン、ファイル名ヘッダーバー、言語バッジ、行番号、行ハイライト、diff表示に対応したシンタックスハイライト付きコードブロックコンポーネントです。

```html
<!-- タイトルと言語指定付きの基本コードブロック -->
<d-codeblock title="Server.js" lang="javascript" copyable>
const port = 3000;
function listen() {
  console.log("listening on", port);
}
</d-codeblock>

<!-- 行ハイライトと行番号表示 -->
<d-codeblock lang="javascript" highlight="2,4-5">
const a = 1;
const b = 2;
const c = 3;
const sum = a + b + c;
console.log(sum);
</d-codeblock>

<!-- フローティングコピーボタン付きdiff表示 -->
<d-codeblock diff copyable>
 initial config
-const timeout = 1000;
+const timeout = 5000;
 return timeout;
</d-codeblock>
```

<a name="属性とプロパティd-codeblock"></a>
## 属性とプロパティ（d-codeblock）

| 属性           | プロパティ     | 型        | デフォルト値 | フォーム | 説明                                                                |
| :------------- | :------------- | :-------- | :----------- | :------- | :------------------------------------------------------------------ |
| `title`        | `title`        | `string`  | `""`         | いいえ   | ヘッダーバーに表示するファイル名またはラベル                        |
| `lang`         | `lang`         | `string`  | `""`         | いいえ   | 大文字バッジとして表示するプログラミング言語識別子                  |
| `code`         | `code`         | `string`  | `""`         | いいえ   | プログラムから指定するソースコード文字列                            |
| `line-numbers` | `lineNumbers`  | `boolean` | `true`       | いいえ   | 各行の横に行番号を表示するかどうか                                  |
| `highlight`    | `highlight`    | `string`  | `""`         | いいえ   | ハイライトする行番号または範囲（例：`"2,4-6"`）                     |
| `diff`         | `diff`         | `boolean` | `false`      | いいえ   | `+`または`-`で始まる行を差分スタイルで表示                          |
| `copyable`     | `copyable`     | `boolean` | `true`       | いいえ   | コピーボタンを表示するかどうか（ヘッダー内または右上に配置）        |
| `wrap`         | `wrap`         | `boolean` | `false`      | いいえ   | 横スクロールの代わりに行を折り返して表示                            |
| `max-height`   | `maxHeight`    | `string`  | `""`         | いいえ   | 長いコードに対する最大スクロール高さ                                |
| `code-font-size` | `codeFontSize` | `string` | `""`         | いいえ   | コードテキスト専用のカスタムフォントサイズ                          |

<a name="プログラム用apid-codeblock"></a>
## プログラム用api（d-codeblock）

```javascript
const block = select('d-codeblock');

// プログラムからクリップボードへコピー
block.copy();

// 生のコードテキストを取得
block.codeText; // string
```

<a name="イベントd-codeblock"></a>
## イベント（d-codeblock）

| イベント | ペイロード         | バブリング | 説明                                           |
| :------- | :----------------- | :--------- | :--------------------------------------------- |
| `copy`   | `{ text: string }` | はい       | コードがクリップボードにコピーされたときに送出 |

<a name="動作仕様d-codeblock"></a>
## 動作仕様（d-codeblock）

- `title`または`lang`が指定されている場合、ファイルアイコン、ファイル名、言語バッジ、コピーボタンを備えたヘッダーバーを描画します。
- `title`と`lang`の双方が省略されている場合、右上にフローティングコピーボタンを配置します。
- コピーボタンをクリックすると、スプリングアニメーションを伴って緑色のチェックマーク（`dstrn-check-line`）に切り替わり、2秒後に元のアイコンへ戻ります。
- ゼロインジェクション仕様の安全な内蔵トークナイザーにより、コードを安全にエスケープして標準トークンクラス（`.d-cb-keyword`、`.d-cb-string`、`.d-cb-number`、`.d-cb-comment`、`.d-cb-function`）を適用します。
- サーバーで事前レンダリングされたマークアップ（`highlight.js`の行要素など）を重複なくシームレスに取り込んで表示します。
