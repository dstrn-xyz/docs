# ビュー

- [ビュー](#ビュー)
  - [はじめに](#はじめに)
  - [ビューの作成](#ビューの作成)
  - [データの表示](#データの表示)
  - [制御構造](#制御構造)
  - [レイアウトとインクルード](#レイアウトとインクルード)
  - [javascriptの実行](#javascriptの実行)
  - [ローカライゼーション](#ローカライゼーション)
    - [日本語ふりがな（ルビ）のサポート](#日本語ふりがな（ルビ）のサポート)
  - [ページネーション](#ページネーション)
    - [カスタムページネーションhtml](#カスタムページネーションhtml)
  - [自動フォーム処理](#自動フォーム処理)
  - [ビューのコンパイル](#ビューのコンパイル)

<a name="はじめに"></a>

## はじめに

dframeworkにはカスタムビューエンジンが搭載されています。テンプレートを生のjavascript関数にコンパイルするstring to function評価モデルを採用しています。これにより一般的なタスクに便利なディレクティブを活用しながらテンプレート内に標準的なjavascriptを直接記述できます。

<a name="ビューの作成"></a>

## ビューの作成

ビューは`views`ディレクトリに保存され`.d`ファイル拡張子を使用する必要があります。ビューはドット記法を使用して参照されます。`views/app/home/index.d`にあるファイルは`app.home.index`として参照されます。

```html
<!-- views/greetings/hello.d -->
<html>
    <body>
        <h1>hello, {{ name }}!</h1>
    </body>
</html>
```

コントローラからは`render()`ヘルパーを使用してビューを返します。

```javascript
export default class GreetingController {
  async hello() {
    return render('greetings.hello', { name: 'world' });
  }
}
```

> [!IMPORTANT]
> `process`、`require`、`global`などの危険なjavascriptグローバル変数はビューデータに渡すことができません。エンジンはビューコンテキストを厳格に検証し改ざんが検出された場合はエラーをスローします。

<a name="データの表示"></a>

## データの表示

ビューに渡されたデータを表示するには変数を二重中括弧で囲みます。ビューエンジンはxss攻撃を防ぐために出力を自動的にエスケープします。

```html
hello, {{ user.name }}.
the current timestamp is {{ Date.now() }}.
```

エスケープされていないhtmlをレンダリングする必要がある場合は`{!! !!}`構文を使用します。

```html
{!! user.bio_html !!}
```

`@json`ディレクティブを使用してオブジェクトをjson文字列に変換できます。`@json`は非同期式のawaitもサポートしています。

```html
<script>
    const userState = @json(user);
    const remoteData = @json(await fetchStats());
</script>
```

<a name="制御構造"></a>

## 制御構造

ビューエンジンは一般的なjavascript制御構造のショートカットを提供します。

**ifステートメント**

```html
@if (user.isAdmin)
    <button>admin panel</button>
@elseif (user.isModerator)
    <button>mod panel</button>
@else
    <span>standard user</span>
@endif
```

**ループ**

```html
@foreach (users as user)
    <li>{{ user.name }}</li>
@endforeach

@for (let i = 0; i < 10; i++)
    <span>{{ i }}</span>
@endfor
```

`@foreach`および`@forelse`ループの内部ではループ変数`$index`（0始まりのインデックス）、`$first`（初回反復時にブール値true）、`$last`（最終反復時にブール値true）が自動的に利用可能です。

`@forelse`ディレクティブは配列が空の場合のフォールバックを備えた配列ループの便利な記述方法です。

```html
@forelse (tracks as track)
    <li>{{ track.title }}</li>
@empty
    <li>no tracks found.</li>
@endforelse
```

<a name="レイアウトとインクルード"></a>

## レイアウトとインクルード

マスターレイアウトを定義し子ビューからそれを拡張できます。コンテンツを注入する場所に`@yield`を指定し子ビューでそのコンテンツを提供するために`@section`を使用します。

```html
<!-- views/layouts/app.d -->
<html>
    <head>
        <title>app</title>
    </head>
    <body>
        <nav></nav>
        <main>
            @yield('content')
        </main>
    </body>
</html>
```

```html
<!-- views/home.d -->
@extends('layouts.app')

@section('content')
    <h1>welcome home</h1>
@endsection
```

ネストされたセクションは最大再帰深度まで解決されます（デフォルトは30、設定の`view.maxYieldDepth`で設定可能）。

`@include`ディレクティブを使用してパーシャルをインクルードできます。ローカル開発環境では存在しないパーシャルを参照するとタイポを迅速に発見できるようにビュー候補を含む実用的な`DiagnosticError`がスローされます。本番環境では存在しないパーシャルは空の文字列として解決されます。

```html
@include('components.header')
```

<a name="javascriptの実行"></a>

## javascriptの実行

ビューエンジンはjavascript関数に直接コンパイルされるためテンプレート内で任意のサーバー側コードを直接実行できます。1行のjavascriptには`@js()`ディレクティブを使用し複数行には`@js`から`@endjs`までのブロックを使用します。

```html
@js(const maxLimit = 50;)

@js
    let sortedUsers = users.sort((a, b) => b.score - a.score);
    sortedUsers = sortedUsers.slice(0, maxLimit);
@endjs

@foreach (sortedUsers as user)
    <li>{{ user.name }}</li>
@endforeach
```

<a name="ローカライゼーション"></a>

## ローカライゼーション

`@t()`ディレクティブを使用して文字列を翻訳できます。フレームワークはリクエストコンテキストから現在のロケールを自動的に解決し対応する翻訳文字列を取得します。`@t()`は非同期の`await t()`呼び出しにコンパイルされるためコンパイル後のビューは非同期として扱われます。

```html
<h1>@t('home.welcome_message')</h1>
<p>@t('home.unread_count', { count: 5 })</p>
```

<a name="日本語ふりがな（ルビ）のサポート"></a>

### 日本語ふりがな（ルビ）のサポート

dframeworkは`@ruby()`ディレクティブを使用した日本語のふりがな（ルビ）アノテーションの組み込みサポートを提供します。2つの引数（漢字と読み）を渡すか`漢字[かんじ]`表記の単一文字列を渡すことができます。`await`を使用した非同期式もサポートされています。

```html
<h1>@ruby("漢字", "かんじ")</h1>
<p>@ruby("日本[にほん]語")</p>
<p>@ruby(await fetchWord())</p>
```

<a name="ページネーション"></a>

## ページネーション

ビューエンジンはモデルの`.paginate()`メソッドによって返されるフレームワークの`Paginator`クラスと統合されています。ペジネーターは`@foreach`ループ内で直接反復可能であり自動またはカスタムのhtmlページネーションコントロールを提供します。

```html
@foreach (users as user)
    <div>{{ user.name }}</div>
@endforeach

<!-- renders standard pagination controls (standalone directive) -->
@pagination(users)
```

あるいはエスケープなしの出力を使用してリンクを直接レンダリングすることもできます:

```html
{!! users.links() !!}
```

検索語やフィルターキーなどの既存のurlクエリパラメータは生成されたすべてのリンクで自動的に保持されます。

デフォルトではリンクはページ全体のナビゲーション用の標準アンカーとしてレンダリングされます。spa遷移を有効にするには第2引数として`true`を渡します:

```html
<!-- enables spa navigation -->
@pagination(users, true)
```

ページ全体の遷移ではなく特定のコンテナを更新するには第3引数としてcssセレクタを渡します:

```html
<!-- updates the #user-list container via spa -->
@pagination(users, true, '#user-list')
```

<a name="カスタムページネーションhtml"></a>

### カスタムページネーションhtml

`@pagination`ブロック内にコンテンツを指定した場合エンジンはデフォルトのhtml生成をスキップしてカスタムマークアップをレンダリングします。

ブロック内ではカスタムコントロールを構築するために使用できる7つの変数がエンジンによって自動的に公開されます:

- `prev`: 前ページのurl（存在しない場合は`null`）
- `next`: 次ページのurl（存在しない場合は`null`）
- `current`: 現在のページ番号
- `last`: 総ページ数
- `total`: 総アイテム数
- `perPage`: 1ページあたりのアイテム数
- `paginator`: ペジネーターインスタンス

```html
@pagination(users)
    <div class="custom-pagination">
        <p>page {{ current }} of {{ last }} ({{ total }} users)</p>
        
        @if (prev)
            <a href="{{ prev }}" class="btn">previous</a>
        @endif
        
        @if (next)
            <a href="{{ next }}" class="btn">next</a>
        @endif
    </div>
@endpagination
```

フレームワークは2ページ以上存在する場合（`last > 1`）にのみブロックを評価します。1ページのみの場合は`@pagination`ブロック全体がスキップされます。

<a name="自動フォーム処理"></a>

## 自動フォーム処理

更新系メソッド（`POST`、`PUT`など）を指定してhtmlの`<form>`を作成するとビューエンジンは隠しフィールド`_csrf`と現在のcsrfトークン値を自動的に注入します。フォーム内で手動でcsrfトークンを出力する必要はありません。

さらにエンジンはドキュメントの`<head>`内に`<meta name="csrf-token">`タグを注入するためajaxリクエストでトークンを取得できます。

<a name="ビューのコンパイル"></a>

## ビューのコンパイル

`local`環境ではフレームワークはカスタムjitコンパイラを使用し元の`.d`ファイルに直接マッピングされた詳細なエラートレースを提供します。

`production`環境ではビューは事前に高度に最適化された`.dc`（.d compiled）ファイルへaot（ahead of time）コンパイルされます。フレームワークは起動時にこれらの`.dc`ファイルの暗号学的完全性を検証しテンプレートが改ざんされていないことを保証します。

コンパイル済みビューは`app.maxCompiledCacheSize`エントリ（デフォルトは`500`）を上限とするLRUキャッシュに保持されます。キャッシュされたビューをレンダリングすると最も最近使用された状態に更新されるため入れ替えが発生しても頻繁に使用されるビューは常駐し続けます。詳細は[設定 > パフォーマンスチューニングキー](../getting-started/configuration.md#config-tuning)を参照してください。
