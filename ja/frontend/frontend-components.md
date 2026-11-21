# フロントエンドコンポーネント

dframework には、フロントエンドバンドルに自動コンパイルされる依存ゼロの高性能カスタムwebコンポーネントライブラリが標準搭載されています。仮想DOM、外部トランスパイル、ビルド手順を必要としない標準カスタム要素として実装されており、サーバー起動時に `public/js/components/` 内のコンポーネントが自動的に検出・バンドルされます。

すべての組み込みコンポーネントは、英語と日本語の辞書を備えたネイティブなクライアント側ローカライズ（`this.t()`）を標準でサポートしています。言語切り替えや文言のカスタマイズ方法については、[多言語化ガイド](../other/localization.md#dcomponent-localization) を参照してください。

## コンポーネント一覧

各コンポーネントは機能ごとに個別のドキュメントファイルに分割されており、構造、ライブ使用例、属性、プログラムAPI、イベント、キーボード操作の仕様が詳しく記載されています。

### レイアウトとナビゲーション

- [`d-sidebar`](./components/d-sidebar.md): オフキャンバス開閉、アイコン縮小、モバイルドロワーに対応したサイドバー。
- [`d-navmenu`](./components/d-navmenu.md): アニメーションインジケーターとビューポートを備えたレスポンシブナビゲーションメニュー。
- [`d-breadcrumb`](./components/d-breadcrumb.md): 中間項目の自動省略とカスタム区切り文字に対応したパンくずリスト。
- [`d-drawer`](./components/d-drawer.md): スワイプ解除や端部固定に対応したスライディングドロワー。
- [`d-hamburger`](./components/d-hamburger.md): 自動階層ドリルダウンを備えたモバイルドロワーメニュー。
- [`d-menubar`](./components/d-menubar.md): 矢印キーナビゲーションに対応したデスクトップアプリケーションメニューバー。
- [`d-link`](./components/d-link.md): モーフィング画面遷移を行う宣言型クライアントルーターリンク。

### フォームと入力

- [`d-text-input`](./components/d-text-input.md): クリアボタンとフローティングラベルを備えたテキスト入力フィールド。
- [`d-combobox`](./components/d-combobox.md): 非同期検索とキーボード操作に対応した絞り込みセレクトボックス。
- [`d-toggle`](./components/d-toggle.md): スライドアクティブインジケーターを備えたセグメント切り替えスイッチ。
- [`d-checkbox`](./components/d-checkbox.md): アニメーションチェックマークを備えた3状態対応チェックボックス。
- [`d-radiobutton`](./components/d-radiobutton.md): キーボードグループ移動に対応した単一選択ラジオボタン。
- [`d-datepicker`](./components/d-datepicker.md): 日付フォーマットとポップアップカレンダーを備えた日付入力。
- [`d-calendar`](./components/d-calendar.md): 単一・範囲・複数日選択に対応したインタラクティブカレンダーグリッド。
- [`d-color-picker`](./components/d-color-picker.md): hex・rgb・hsl入力に対応した視覚的カラーピッカー。
- [`d-file-input`](./components/d-file-input.md): プログレス表示を備えたドラッグ＆ドロップファイル入力。
- [`d-image-input`](./components/d-image-input.md): プレビューとアスペクト比固定に対応した画像アップロード入力。
- [`d-slider`](./components/d-slider.md): ツールチップバブルとステップスナップを備えた数値スライダー。
- [`d-otp`](./components/d-otp.md): 自動フォーカス移動とクリップボード貼り付けに対応したワンタイム認証コード入力。
- [`d-form`](./components/d-form.md): 自動エラー処理と読み込み状態管理を備えたリアクティブajaxフォーム。

### オーバーレイとフィードバック

- [`d-modal`](./components/d-modal.md): バックドロップ解除とエスケープキーに対応したダイアログオーバーレイ。
- [`d-popover`](./components/d-popover.md): ビューポート自動位置調整を備えたアンカー固定ポップオーバー。
- [`d-notification`](./components/d-notification.md): スタック表示と自動消去タイマーを備えたトースト通知。
- [`d-tooltip`](./components/d-tooltip.md): ホバーおよびフォーカス時にショートカットバッジを表示するツールチップ。
- [`d-context-menu`](./components/d-context-menu.md): 階層サブメニューに対応した右クリックコンテキストメニュー。
- [`d-dropdown`](./components/d-dropdown.md): キーボード選択に対応したドロップダウントリガーおよびメニュー。

### データと表示

- [`d-codeblock`](./components/d-codeblock.md): 行番号とコピーボタンを備えたシンタックスハイライトコードブロック。
- [`d-skeleton`](./components/d-skeleton.md): カスタム寸法に対応したコンテンツ読み込みプレースホルダー。
- [`d-progress`](./components/d-progress.md): 確定および不確定モードに対応した進捗インジケーター。
- [`d-virtual-list`](./components/d-virtual-list.md): 大規模データセット向けの高パフォーマンス仮想スクローラー。
- [`d-key`](./components/d-key.md): OS別キーボードシンボル自動変換に対応したショートカットバッジ。
- [`d-loader`](./components/d-loader.md): 非同期状態を示すスピナーおよびパルスアニメーションローダー。

### アクションとコントロール

- [`d-buttongroup`](./components/d-buttongroup.md): 共通角丸と境界線を備えたボタン連結ツールバー。
- [`d-hold-button`](./components/d-hold-button.md): 円形プログレスリングによる長押し確認ボタン。
- [`d-icon-button`](./components/d-icon-button.md): アイコンフォントに最適化された正方形アクセシブルボタン。
- [`d-morph`](./components/d-morph.md): 要素間のスムーズな形状変化を実現するモーフィングコンテナ。
- [`d-questionnaire`](./components/d-questionnaire.md): フォーム検証を備えた複数ステップ対話ウィザード。

## 組み込みコンポーネントのオーバーライド

組み込みコンポーネントのすべての動作はアプリケーションコードによってカスタマイズ、拡張、または完全に上書きできます。組み込みコンポーネントをオーバーライドするには`public/js/components/`配下にまったく同じコンポーネントファイル名でファイルを配置します（例: `public/js/components/dCheckbox.js`）。

フロントエンドバンドルのコンパイル時にコンパイラはユーザーファイルを自動検出し組み込みコンポーネントの実装をその場で置き換えます。

```javascript
class dCheckbox extends dComponent {
  static form = true;
  static get tag() { return 'd-checkbox'; }
}

dComponent.define(dCheckbox);
```

<a name="イミュータブルな基底クラス"></a>

### イミュータブルな基底クラス

すべてのカスタム要素およびオーバーライドは`dComponent`を継承します。UIコンポーネントとは異なり`dComponent`はリアクティビティ、リフレクション、ライフサイクル管理を提供するイミュータブルなフレームワーク基底クラスです。コンパイラは常にバンドルのルートにコアの`dComponent`実装を保持するため、すべてのコンポーネントが一貫してこれを継承します。

---

<a name="dComponent（基底クラス）"></a>

## dComponent（基底クラス）

`dComponent`はdframeworkにおけるすべてのカスタムwebコンポーネントのオーサリング基底クラスです。プロパティリフレクション、リアクティブプロキシステート、フォーム統合、局所的DOMレンダリング、自動メモリクリーンアップを提供します。

```javascript
class dCounter extends dComponent {
  static tag = 'd-counter';
  static form = true;

  static props = {
    value: { type: 'number', default: 0, form: true },
    step: { type: 'number', default: 1 },
    disabled: { type: 'boolean', default: false },
    label: { type: 'string', default: 'count' }
  };

  template() {
    return `
      <span class="label">${this.label}</span>
      <button class="dec" type="button">−</button>
      <span class="value">${this.value}</span>
      <button class="inc" type="button">+</button>
    `;
  }

  mount() {
    this.effect(() => {
      document.title = `count: ${this.state.value ?? this.value}`;
      return () => { document.title = 'app'; };
    });
  }

  render() {
    this.refs('.value')[0].textContent = this.state.value ?? this.value;
    this.refs('button.dec')[0].disabled = this.disabled;
    this.refs('button.inc')[0].disabled = this.disabled;

    this.listen(this.refs('button.dec')[0], 'click', () => this.decrement());
    this.listen(this.refs('button.inc')[0], 'click', () => this.increment());
  }

  onPropChanged(name, oldVal, newVal) {
  }

  increment() {
    this.setState(s => { s.value = (s.value ?? this.value) + this.step; });
  }

  decrement() {
    this.setState(s => { s.value = (s.value ?? this.value) - this.step; });
  }
}

dComponent.define(dCounter);
```

<a name="コンポーネントの定義と登録"></a>

### コンポーネントの定義と登録

`dComponent.define(Class)`を使用してコンポーネントを登録します:

```javascript
dComponent.define(dCounter);
```

- 型変換（`number`、`boolean`、`string`、`object`、`array`）を伴うゲッターおよびセッタープロパティをプロトタイプ上に自動定義します。
- 監視対象属性をプロパティセッターにバインドします。
- `Class.tag`を使用して`customElements.define`に登録します。

<a name="ライフサイクルメソッド"></a>

### ライフサイクルメソッド

| メソッド                        | タイミング                | 説明                                                |
| :------------------------------ | :------------------------ | :-------------------------------------------------- |
| `template()`                    | マウント前                | 内部DOMを構築するための初期HTML文字列を返却         |
| `mount()`                       | 初回接続時                | コンポーネントロジックを初期化しエフェクトを登録    |
| `mountSync()`                   | 手動/即時                 | RAF遅延を待たずに即時同期マウントを実行             |
| `connected()`                   | 接続ごと                  | 要素がドキュメントにアタッチされるたびに呼び出し    |
| `render()`                      | ステート/プロパティ変更時 | 局所的なDOMノード更新を実行                         |
| `onPropChanged(name, old, new)` | プロパティ変更時          | 特定のプロパティの変更に反応                        |
| `syncForm()`                    | フォーム/手動             | ElementInternalsのフォーム値を同期（独自ルールでオーバーライド可能） |
| `onscreen()`                    | ビューポート進入時        | コンポーネントがビューポートに入ったときに呼び出し  |
| `offscreen()`                   | ビューポート退出時        | コンポーネントがビューポートから出たときに呼び出し  |
| `destroy()`                     | 切断時                    | 要素がDOMから削除されたときに呼び出される破棄フック |

> [!NOTE]
> `mount()`の初期化は初回DOM接続時に`requestAnimationFrame`を介してバッチ処理されます。これによりライトDOM内で解析されたすべての子ノードが`mount()`実行前に存在し利用可能であることが保証されます。プログラムから動的に子カスタム要素を生成した場合や、生成直後に即座に同期初期化が必要な場合は、`element.mountSync()`または`dComponent.mountSync(element)`を呼び出します。テンプレートコンポーネントの場合テンプレートマークアップは定義時に事前解析されキャッシュされた`cloneNode(true)`を介して適用されます。その後のプロパティおよびステートの更新は静的な3フェーズマイクロタスクフラッシュにバッチ処理されます。

<a name="フラグメントキャッシュと一括マウント"></a>

### フラグメントキャッシュと一括マウント

動的コンテンツの生成、内部マークアップの設定、またはコンポーネントリストのマウント時において、最大のスループットとXSS安全性を確保するために、`dComponent`はテンプレートキャッシュユーティリティと安全なDOM置換機能を提供します:

```javascript
const frag = dComponent.fragment('<div class="list-item"><span>title</span></div>');
dComponent.appendMany(container, frag1, frag2, '<div class="extra">item</div>');
this.setChildren('<div class="card"><span class="title"></span></div>');
```

<a name="レンダリング戦略と局所的更新"></a>

### レンダリング戦略と局所的更新

`dComponent`は仮想DOMのオーバーヘッドを回避します。DOMノードは`template()`または`mount()`で一度だけ作成され動的な更新は`render()`内で局所的に適用されます。

> [!IMPORTANT]
> `render()`内で`this.state`を変更してはいけません。無限更新ループを防止するため`render()`内でのステート変更はブロックされます。

<a name="バッチ化マイクロタスク更新キュー"></a>

### バッチ化マイクロタスク更新キュー

プロパティの変更とステートの変更はすべてのアクティブコンポーネントにわたって自動的にまとめられ、静的な3フェーズマイクロタスクフラッシュに統合されます:

1. **フェーズ1（属性同期）:** 変更されたプロパティの監視対象HTML属性を同期します。
2. **フェーズ2（レンダリング）:** 一時的なレンダーリスナーをクリアし更新されたコンポーネントごとに`render()`を一度実行します。
3. **フェーズ3（エフェクトとフォーム同期）:** リアクティブエフェクトを再評価しrefsキャッシュを無効化しフォーム内部を同期して集約されたdebugbarイベントを送出します。

```javascript
this.state.title = 'new title';
this.state.count = 42;
```

<a name="リアクティブステート"></a>

### リアクティブステート

```javascript
this.setState({ active: true });
this.setState(s => { s.count++; s.updatedAt = Date.now(); });
this.setState({ timer: Date.now() }, false);
```

<a name="エフェクトと依存関係追跡"></a>

### エフェクトと依存関係追跡

```javascript
this.effect(() => {
  console.log('value updated:', this.state.value);
  return () => console.log('cleaning up effect');
});

this.effect(() => {
  fetchData(this.query);
}, () => [this.query]);
```

<a name="自動クリーンアップapi"></a>

### 自動クリーンアップapi

これらのメソッドはコンポーネントがDOMから削除されたときにリスナー、タイマー、およびアニメーションフレーム要求を自動的にクリーンアップします:

```javascript
const off = this.listen(element, 'click', (e) => this.handleClick(e));
off();

const offAll = this.listenAll(elements, 'click', (e) => this.handleClick(e));
offAll();

this.unlisten(element, 'click', handler);

this.setTimeout(() => this.tick(), 1000);
this.setInterval(() => this.refresh(), 5000);

this.requestAnimationFrame((time) => this.draw(time));
```

<a name="domキャッシュとrefs"></a>

### domキャッシュとrefs

`this.refs(selector)`はレンダリング間でクエリされた要素配列をキャッシュし、繰り返されるDOMクエリのオーバーヘッドを排除します:

```javascript
const [inputEl] = this.refs('input.search');
```

<a name="内部コンテンツの取得"></a>

### 内部コンテンツの取得

`template()`実行前にカスタム要素に渡された元のライトDOMノードにアクセスします:

```javascript
this.originalChildren
this.originalHTML
this.cloneChildren()
```

<a name="フォーム統合"></a>

### フォーム統合

`static form = true`の場合、`dComponent`は親`<form>`要素と統合され`form: true`とマークされたプロパティを送信します:

```javascript
static form = true;
static props = {
  value: { type: 'string', default: '', form: true }
};

console.log(this.form);
```

単一の値、ファイルインスタンス（`File`）、ファイル配列（`File[]`）、および複数のフォームプロパティを自動的にサポートします。

独自のフォーム送信ロジック（未チェック状態のチェックボックスやラジオボタンの除外、無効化時の値のクリアなど）を実装する場合は、`syncForm()`をオーバーライドします:

```javascript
syncForm() {
  if (!this._internals) return;
  if (this.checked && !this.disabled) {
    this._internals.setFormValue(this.value || 'on');
  } else {
    this._internals.setFormValue(null);
  }
}
```

<a name="属性リフレクションとプログラムによる更新"></a>

### 属性リフレクションとプログラムによる更新

再帰的な`attributeChangedCallback`のトリガーを回避しながら安全にホストDOM要素へ属性を即時反映するには、`this.reflectAttribute(name, value)`を使用します:

```javascript
// ホスト要素に安全に属性を即時反映
this.reflectAttribute('placement', 'before');
```

プロパティの更新がスクリプトや`onPropChanged()`を介したプログラムによる更新かどうかを判定するには、`this.isProgrammaticUpdate`を参照します:

```javascript
if (this.isProgrammaticUpdate) return;
```

<a name="内部イベントエミッター"></a>

### 内部イベントエミッター

```javascript
const off = this.on('custom-event', (data) => console.log(data));
off();

this.off('custom-event', handler);

this.emit('custom-event', { id: 123 });
```

<a name="websocket統合"></a>

### websocket統合

```javascript
this.wire('chat:send', { message: 'hello' });
```

