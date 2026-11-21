# d-questionnaire

進捗バー、ラジオボタン、チェックボックス、テキスト入力、キーボードショートカットを統合した対話型マルチステップアンケートコンポーネントです。

```html
<d-questionnaire name="survey">
  <div d-step name="role" title="役割を選択してください" type="single" required>
    <div d-option value="dev" text="開発者" description="バックエンドおよびソフトウェア構築"></div>
    <div d-option value="design" text="デザイナー" description="ユーザーインターフェース設計"></div>
  </div>
  <div d-step name="tools" title="使用しているツール" type="multiple" skippable>
    <div d-option value="git" text="git"></div>
    <div d-option value="docker" text="docker"></div>
  </div>
  <div d-step name="notes" title="ご意見・ご要望" type="text" placeholder="自由に入力してください"></div>
</d-questionnaire>
```

<a name="属性とプロパティd-questionnaire"></a>
## 属性とプロパティ（d-questionnaire）

| 属性 | プロパティ | 型 | デフォルト | フォーム | 説明 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `name` | `name` | string | `""` | いいえ | フォームフィールド名 |
| `value` | `value` | string | `"{}"` | はい | 回答データのjsonシリアライズ文字列（フォーム連動） |
| `current-step` | `currentStep` | number | `0` | いいえ | 現在のアクティブステップ番号 |
| `auto-advance` | `autoAdvance` | boolean | `false` | いいえ | 単一選択肢クリック時に自動進行するかどうか |
| `show-progress` | `showProgress` | boolean | `true` | いいえ | 上部進捗バーの表示 |
| `show-shortcuts` | `showShortcuts` | boolean | `true` | いいえ | 選択肢の数字キーバッジ表示 |
| `questions` | `questions` | string | `""` | いいえ | 子要素の代わりにjson文字列でステップを定義 |

<a name="プログラム用apid-questionnaire"></a>
## プログラム用api（d-questionnaire）

```javascript
const qn = select('d-questionnaire');

// ナビゲーション操作
qn.next();             // 入力検証後に次のステップへ進むか送信
qn.prev();             // 前のステップへ戻る
qn.skip();             // 任意項目のステップをスキップ
qn.goTo(1);            // 指定のステップインデックスへ移動
qn.reset();            // 回答をクリアして初期ステップへ戻す

// 回答データ取得と設定
const ans = qn.answers;              // 収集された回答オブジェクトのコピー
qn.setAnswer('role', 'dev');        // プログラムから回答を設定
```

<a name="イベントd-questionnaire"></a>
## イベント（d-questionnaire）

| イベント | 詳細 | バブリング | 説明 |
| :--- | :--- | :--- | :--- |
| `step` | オブジェクト（`{ step, index, total, answers }`） | はい | ステップ移動時に送出 |
| `answer` | オブジェクト（`{ name, value, answers }`） | はい | 回答が選択・変更されたときに送出 |
| `submit` | オブジェクト（`{ answers, json }`） | はい | 最終ステップ完了時に送出 |

<a name="動作仕様d-questionnaire"></a>
## 動作仕様（d-questionnaire）

- キーボード操作に対応（数字キー`1`〜`9`で該当選択肢を選択、`Enter`で次へ、`Backspace`または`ArrowLeft`で戻る）。
- 必須ステップ未回答時はシェイクアニメーションを実行して進行を防止。
- 回答はjson文字列として自動更新され、`<form>`や`<d-form>`の送信データに含まれます。
