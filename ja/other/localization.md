# 多言語化

- [多言語化](#多言語化)
  - [はじめに](#はじめに)
  - [翻訳の定義](#翻訳の定義)
    - [ディレクトリ構成](#ディレクトリ構成)
    - [翻訳ファイル](#翻訳ファイル)
    - [ネストされたキー](#ネストされたキー)
  - [翻訳の取得](#翻訳の取得)
    - [tヘルパー](#tヘルパー)
    - [ビュー内での使用](#ビュー内での使用)
    - [フォールバック動作](#フォールバック動作)
    - [変数補間](#変数補間)
    - [日本語のルビ（ふりがな）サポート](#日本語のルビ（ふりがな）サポート)
  - [ロケールの検出](#ロケールの検出)
    - [検出順序](#検出順序)
    - [デフォルトロケール](#デフォルトロケール)
  - [ロケールの設定](#ロケールの設定)
    - [永続的なロケール](#永続的なロケール)
    - [リクエスト限定のロケール](#リクエスト限定のロケール)
    - [ロケールの読み取り](#ロケールの読み取り)
  - [キャッシングとホットリロード](#キャッシングとホットリロード)
  - [dComponentの多言語化](#dcomponentの多言語化)
    - [コンポーネント翻訳の定義](#component-translations)
    - [コンポーネント内での翻訳](#コンポーネント内での翻訳)
    - [ロケール解決の順序](#コンポーネントのロケール解決順序)
    - [実行時のロケール切り替え](#実行時のロケール切り替え)
    - [翻訳辞書の登録](#翻訳辞書の登録)
    - [ロケールの読み取りと静的ヘルパー](#静的ヘルパー)

<a name="はじめに"></a>

## はじめに

dframeworkは多言語アプリケーションを構築するための軽量な多言語化システムを提供します。翻訳はロケールごとに整理されたjsonファイルとして保存されフレームワークはコントローラーとビューの両方で翻訳された文字列を取得するためのヘルパーを提供します。ロケール検出はクッキーおよび`Accept-Language`ヘッダーから自動的に行われアクティブなロケールはリクエストごとに変更するかユーザーのセッションに永続化できます。

<a name="翻訳の定義"></a>

## 翻訳の定義

<a name="ディレクトリ構成"></a>

### ディレクトリ構成

翻訳ファイルはプロジェクトルートの`lang`ディレクトリに保存されます。各ロケールにはロケールコードという名前の独自のサブディレクトリがあります。各ロケールディレクトリ内に、関連する翻訳キーをグループ化したjsonファイルを作成します。

```
lang/
├── en/
│   ├── common.json
│   ├── auth.json
│   └── dashboard.json
├── ja/
│   ├── common.json
│   ├── auth.json
│   └── dashboard.json
└── es/
    ├── common.json
    └── auth.json
```

<a name="翻訳ファイル"></a>

### 翻訳ファイル

各翻訳ファイルはキーを翻訳された文字列にマッピングする標準的なjsonオブジェクトです。

```json
{
  "welcome": "welcome to dframework",
  "goodbye": "see you next time",
  "greeting": "hello, welcome back"
}
```

```json
{
  "welcome": "dframeworkへようこそ",
  "goodbye": "またお会いしましょう",
  "greeting": "こんにちは、おかえりなさい"
}
```

<a name="ネストされたキー"></a>

### ネストされたキー

翻訳ファイルはネストされたオブジェクトをサポートします。ドット記法を使用してネストされたキーにアクセスします。

```json
{
  "buttons": {
    "save": "save changes",
    "cancel": "cancel",
    "delete": "delete permanently"
  },
  "messages": {
    "success": "operation completed",
    "error": "something went wrong"
  }
}
```

```javascript
await t(req, 'common.buttons.save');
// "save changes"
```

<a name="翻訳の取得"></a>

## 翻訳の取得

<a name="tヘルパー"></a>

### tヘルパー

`t()`関数は翻訳を取得する主要な方法です。グローバルに利用可能です。ドット記法のキーで直接呼び出すかリクエストオブジェクトを明示的に渡すことができます。リクエストオブジェクトを省略した場合`t()`は非同期リクエストコンテキストから現在のリクエストを自動的に解決します。

```javascript
// auto resolves request from context
const welcome = await t('common.welcome');
const missing = await t('common.nonexistent', 'default text');

// or pass request explicitly
const welcomeExplicit = await t(req, 'common.welcome');
```

キーの最初のセグメントはファイル名（`.json`拡張子なし）であり残りのセグメントはjsonオブジェクト内のパスを形成します。

| キー | ファイル | パス |
| ----------------------- | ------------------------------ | ---------------- |
| `common.welcome`        | `lang/{locale}/common.json`    | `welcome`        |
| `auth.errors.invalid`   | `lang/{locale}/auth.json`      | `errors.invalid` |
| `dashboard.stats.users` | `lang/{locale}/dashboard.json` | `stats.users`    |

最初のアクセス時にディスクから翻訳ファイルを読み取るためこの関数は非同期です。

<a name="ビュー内での使用"></a>

### ビュー内での使用

ビューテンプレートでは`@t()`ディレクティブを使用します。フレームワークはリクエストコンテキストから現在のロケールを自動的に解決します。

```html
<h1>@t('common.welcome')</h1>
<button>@t('common.buttons.save')</button>
```

`@t()`ディレクティブは現在のリクエストオブジェクトを使用して非同期の`await t()`呼び出しにコンパイルされる構文糖です（コンパイル後のテンプレートは非同期として扱われます）。手動でリクエストを渡す必要はありません。

<a name="フォールバック動作"></a>

### フォールバック動作

> [!NOTE]
> 翻訳キーが存在せずフォールバック文字列も指定されていない場合`t()`は空のテキストではなく生のキーパスを返します。

```javascript
await t(req, 'common.missing_key');
// returns "common.missing_key"

await t(req, 'common.missing_key', 'fallback text');
// returns "fallback text"
```

現在のロケールに対応する翻訳ファイル自体が存在しない場合も関数は適切にフォールバックします。

<a name="変数補間"></a>

### 変数補間

翻訳はフレームワークのルートパラメータ規約に一致する`:name`構文を使用した動的パラメータプレースホルダーをサポートします。オブジェクトを`t()`の第3引数または`@t()`の第2引数として渡します。

```json
{
  "greeting": "こんにちは、:nameさん！",
  "items": ":name様は:count個のアイテムを持っています。"
}
```

```javascript
await t(req, 'messages.greeting', { name: '太郎' });
// "こんにちは、太郎さん！"

await t(req, 'messages.items', { name: '山田', count: 5 });
// "山田様は5個のアイテムを持っています。"
```

ビューではパラメータオブジェクトを`@t()`に渡します:

```html
<p>@t('messages.greeting', { name: '太郎' })</p>
```

<a name="日本語のルビ（ふりがな）サポート"></a>

### 日本語のルビ（ふりがな）サポート

dframeworkは日本語のルビ（ふりがな）注釈の組み込みサポートを提供します。`漢字[かんじ]`表記を使用して翻訳文字列内に直接ふりがなを定義するかグローバルの`ruby()`ヘルパーおよび`@ruby()`ディレクティブを使用できます。

```json
{
  "welcome": "東京[とうきょう]へようこそ、:nameさん！"
}
```

```javascript
await t(req, 'messages.welcome', { name: '花子' });
// "<ruby>東京<rt>とうきょう</rt></ruby>へようこそ、花子さん！"
```

また`ruby()`を直接呼び出すかビューで`@ruby()`を使用することもできます:

```javascript
ruby('漢字', 'かんじ');
// "<ruby>漢字<rt>かんじ</rt></ruby>"

ruby('日本[にほん]語[ご]へようこそ');
// "<ruby>日本<rt>にほん</rt></ruby><ruby>語<rt>ご</rt></ruby>へようこそ"
```

```html
<h1>@ruby("漢字", "かんじ")</h1>
<p>@ruby("日本[にほん]語")</p>
```

`ruby()`とふりがな注釈付きの翻訳はどちらもビューでレンダリングされたときに安全なhtmlを出力します。

<a name="ロケールの検出"></a>

## ロケールの検出

<a name="検出順序"></a>

### 検出順序

dframeworkは以下の優先順位を使用してユーザーのロケールを自動的に検出します:

1. **localeクッキー**（`locale`クッキーが存在し有効なロケールコード（2〜10文字の英数字）が含まれている場合、即座に使用されます）
2. **Accept-Languageヘッダー**（クッキーが見つからない場合、フレームワークは`Accept-Language` httpヘッダーを解析し、`lang/`でサポートされている場合は地域ロケールコード（例: `en-US`、`zh-CN`）に一致させ、そうでない場合は基本言語コード（`en`、`zh`）にフォールバックします）
3. **デフォルトロケール**（どちらのソースも有効なロケールを提供しない場合、設定されたデフォルトロケールが使用されます）

この検出はリクエスト処理中に自動的に行われます。検出されたロケールはリクエストオブジェクト上に`req.locale`として保存されます。

<a name="デフォルトロケール"></a>

### デフォルトロケール

デフォルトロケールは`config/app.js`ファイルの`locale`キーで設定されます。

```javascript
// config/app.js
export default {
  locale: 'en'
};
```

デフォルトが設定されていない場合フレームワークは`'en'`にフォールバックします。

<a name="ロケールの設定"></a>

## ロケールの設定

<a name="永続的なロケール"></a>

### 永続的なロケール

ユーザーのロケールを変更するには`Locale.set()`関数を使用します。デフォルトでは`locale`クッキーを書き込みユーザーのセッションに設定を保存することで変更が永続化されます。`Locale.set()`はpromiseを返しawaitできます。

```javascript
import { Locale } from 'dframework';

await Locale.set('ja');
```

クッキーは1年間の有効期間（秒単位で`maxAge: 365 * 24 * 60 * 60`）で書き込まれるためユーザーの言語設定はブラウザセッションをまたいで保持されます。クッキーは`SameSite=Lax`、非local環境での`secure`フラグ、およびルートパス`/`を使用します。

ロケール値は標準のBCP 47形式（`/^[a-zA-Z]{2,8}(-[a-zA-Z0-9]{2,8})*$/`）に一致する必要があり`es-419`、`zh-Hans-CN`、`zh-Hant-TW`などの地域タグや数値識別子をサポートします。無効な値はエラーをスローします。

<a name="リクエスト限定のロケール"></a>

### リクエスト限定のロケール

永続化せずに現在のリクエストに対してのみロケールを変更する必要がある場合は第2引数として`true`を渡します。

```javascript
Locale.set('fr', true);
```

これによりリクエストの期間中`req.locale`が設定されますがクッキーの書き込みやセッションの更新は行われません。これは異なる言語でのコンテンツのプレビューやロケールパラメータを受け入れるapiエンドポイントに役立ちます。

<a name="ロケールの読み取り"></a>

### ロケールの読み取り

リクエストコンテキストから現在のロケールを読み取ることができます。`Locale`エクスポートはロケール管理メソッドへのアクセスを提供します。

```javascript
import { Locale } from 'dframework';

const currentLocale = Locale.get();
```

デフォルトロケールを直接読み取ることもできます。

```javascript
const defaultLocale = Locale.getDefault();
```

<a name="キャッシングとホットリロード"></a>

## キャッシングとホットリロード

翻訳ファイルは最初の読み取り後にメモリにキャッシュされます。同じロケールとファイルの組み合わせでの以降の`t()`呼び出しはファイルシステムにアクセスすることなくキャッシュされた辞書を返します。

`local`環境ではフレームワークはロードされた各翻訳ファイルにファイルウォッチャーを設定します。翻訳ファイルを編集して保存するとその特定のファイルとロケールのキャッシュが自動的に無効化されます。次回の`t()`呼び出しで更新されたファイルがディスクから再読み込みされます。これによりサーバーを再起動することなく開発中に即座にフィードバックが得られます。

ファイルウォッチャーはアプリケーション終了時に自動的にクリーンアップされます。本番環境ではファイルウォッチャーは作成されずキャッシュはプロセスの存続期間中持続します。

<a name="dcomponentの多言語化"></a>

## dComponentの多言語化

すべての組み込みおよびカスタムdComponentは`dComponent`を介した統一的なクライアントサイド多言語化に対応しています。主要コンポーネントはネットワーク通信なしで独自の英語と日本語辞書をカプセル化して標準搭載しています。

<a name="コンポーネント固有の翻訳定義"></a>

### コンポーネント固有の翻訳定義

各コンポーネントは静的ゲッターを使用して独自のデフォルト翻訳を直接カプセル化します。これにより単一責任の原則が維持され基底クラスが肥大化しません。

```javascript
class MyUploader extends dComponent {
  static get tag() { return "my-uploader"; }

  static get translations() {
    return {
      en: {
        select_files: "choose files to upload",
        selected_count: ":count files selected"
      },
      ja: {
        select_files: "アップロードするファイルを選択",
        selected_count: ":count個のファイルが選択されました"
      }
    };
  }

  mount() {
    const label = this.t("select_files", "アップロードするファイルを選択");
    const counter = this.t("selected_count", { count: 3 }, ":count件のファイルを選択中");
  }
}
```

`dComponent.define()`でコンポーネントを定義すると宣言された翻訳辞書がコンポーネントのタグ名に対して自動登録されます。

<a name="コンポーネント内での翻訳"></a>

### コンポーネント内での翻訳

任意の`dComponent`サブクラス内で`this.t(key, params, fallback)`を使用して翻訳文字列を取得できます。翻訳キーは以下の順序で解決されます。

1. `dComponent.setTranslations()`で登録されたグローバルユーザー設定
2. クラス上で定義されたコンポーネント固有の翻訳 (`static get translations`)
3. タグ名に対して登録されたコンポーネント辞書
4. フォールバック値または生のキー文字列

キーは`:name`形式のパラメータ補間に対応しています。

<a name="コンポーネントのロケール解決順序"></a>

### ロケール解決の順序

各コンポーネントは以下の階層順序でアクティブなロケールを決定します。

1. コンポーネントの明示的な属性またはプロパティ (`<d-calendar locale="ja">`)
2. `lang`または`locale`属性を持つ最も近い親要素 (`<div lang="ja"><d-file-input></div>`)
3. `dComponent.setLocale()`で設定されたグローバルクライアントロケール
4. ルートhtml要素の言語属性 (`document.documentElement.lang`)
5. ブラウザの言語設定
6. デフォルトのフォールバックロケール (`en`)

<a name="実行時のロケール切り替え"></a>

### 実行時のロケール切り替え

`dComponent.setLocale()`を呼び出すとアクティブなロケールが更新されページのリロードなしで画面上のすべてのマウント済みコンポーネントが即座に再レンダリングされます。

```javascript
// 言語を切り替え、表示中のすべてのdComponentを即座に更新
dComponent.setLocale("ja");
```

<a name="翻訳辞書の登録"></a>

### 翻訳辞書の登録

外部言語パックの読み込みや既存の翻訳の上書きには`dComponent.setTranslations()`を使用します。

```javascript
dComponent.setTranslations("fr", {
  "d-file-input": {
    placeholder: "déposez le fichier ici",
    subtitle: "ou cliquez pour parcourir"
  }
});
```

<a name="静的ヘルパー"></a>

### ロケールの読み取りと静的ヘルパー

コンポーネントインスタンスの外部でも、`dComponent`の静的ヘルパーを使用してアクティブなロケールを取得したり、文字列を直接翻訳できます。

```javascript
// アクティブなクライアントロケールコードを取得 (例: "ja", "en-US")
const active = dComponent.getLocale();

// 文字列を直接翻訳
const label = dComponent.t("common.save", "保存");
const welcome = dComponent.t("welcome.user", { name: "alice" }, "こんにちは :name");
```