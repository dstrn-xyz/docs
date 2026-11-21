# d-navmenu

アニメーション化されたホバーピル、方向連動のスライド遷移を備えた共有モーフィングビューポートパネル、およびモバイル時の`d-hamburger`とのシームレスなドリルダウン連携を特徴とするウェブサイト用水平ナビゲーションメニューです。

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <d-navmenu id="site-nav">
      <a d-link href="/" class="nav-link">overview</a>
      <div d-submenu="getting started">
        <span slot="trigger">getting started</span>
        <div slot="content" class="flex-column g-05" style="width: 280px;">
          <a d-link href="/docs" class="d-navmenu-sublink">
            <span class="d-navmenu-sublink-title">introduction</span>
            <span class="d-navmenu-sublink-desc">reusable frontend component library</span>
          </a>
          <a d-link href="/installation" class="d-navmenu-sublink">
            <span class="d-navmenu-sublink-title">installation</span>
            <span class="d-navmenu-sublink-desc">quick start and directory structure</span>
          </a>
        </div>
      </div>
      <div d-submenu="components">
        <span slot="trigger">components</span>
        <div slot="content" class="flex-row g-1" style="width: 380px;">
          <div class="flex-column g-05 w-50">
            <a d-link href="/popover" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">popover</span>
              <span class="d-navmenu-sublink-desc">top layer floating overlay</span>
            </a>
            <a d-link href="/tooltip" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">tooltip</span>
              <span class="d-navmenu-sublink-desc">hover hint with shortcut badge</span>
            </a>
          </div>
          <div class="flex-column g-05 w-50">
            <a d-link href="/menubar" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">menubar</span>
              <span class="d-navmenu-sublink-desc">desktop application bar</span>
            </a>
            <a d-link href="/otp" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">otp</span>
              <span class="d-navmenu-sublink-desc">one time verification code</span>
            </a>
          </div>
        </div>
      </div>
      <a d-link href="/docs" class="nav-link">documentation</a>
    </d-navmenu>

    <!-- モバイル向けハンバーガー連携：d-navmenuの項目をドリルダウンパネルへ自動変換 -->
    <d-hamburger target="#site-nav" breakpoint="md">
      <div slot="header">
        <span class="fs-09 text-content-l">menu</span>
      </div>
    </d-hamburger>
  </div>
</nav>
```

<a name="属性とプロパティd-navmenu"></a>
## 属性とプロパティ（d-navmenu）

| 属性          | プロパティ    | 型        | デフォルト値   | フォーム | 説明                                                                             |
| :------------ | :------------ | :-------- | :------------- | :------- | :------------------------------------------------------------------------------- |
| `items`       | `items`       | `array`   | `null`         | いいえ   | 構造化アイテム配列（宣言的子要素の代替）                                         |
| `delay`       | `delay`       | `number`  | `150`          | いいえ   | メニューが閉じるまでのホバー猶予遅延ミリ秒                                       |
| `orientation` | `orientation` | `string`  | `"horizontal"` | いいえ   | メニューのレイアウト方向（`"horizontal"`, `"vertical"`）                         |
| `align`       | `align`       | `string`  | `"center"`     | いいえ   | アクティブトリガーに対するビューポートの整列（`"center"`, `"start"`, `"end"`）   |
| `disabled`    | `disabled`    | `boolean` | `false`        | いいえ   | ナビゲーションの操作を無効化                                                     |

宣言的マークアップ属性：

- `d-submenu="<title>"`: ドロップダウンパネルコンテナー（`<d-navmenu>`の子要素）
- `slot="trigger"`: `[d-submenu]`内のトリガーラベルまたはマークアップ
- `slot="content"`: ドロップダウンパネルの内容ラッパー
- `.d-navmenu-sublink`: `.d-navmenu-sublink-title`および`.d-navmenu-sublink-desc`を含むカードリンク

<a name="プログラム用apid-navmenu"></a>
## プログラム用api（d-navmenu）

```javascript
const nav = select('d-navmenu');

nav.open(1);
nav.close();
nav.isOpened;
nav.activeIndex;

// モバイルメニュー用に元の要素クローンを取得
const items = nav.getHamburgerItems();

// すべてのnavmenuインスタンスを閉じる
dNavMenu.closeAll();
```

<a name="イベントd-navmenu"></a>
## イベント（d-navmenu）

| イベント | 詳細                                | バブリング | 説明                                 |
| :------- | :---------------------------------- | :--------- | :----------------------------------- |
| `open`   | `{ index, title, item }`            | はい       | サブメニューが開いたときに送出       |
| `close`  | なし                                | はい       | アクティブなサブメニューが閉じたときに送出 |

<a name="動作仕様とd-hamburger連携"></a>
## 動作仕様とd-hamburger連携

- 共有モーフィングビューポートパネルが幅と高さを滑らかにアニメーションさせながら、タブ間の移動方向に応じたクロススライド（進行方向への`translateX`）でコンテンツを切り替えます。
- アニメーション化されたホバーピルがアクティブまたはホバーされたトリガーの背後を追従します。
- ビューポートの出現には`@starting-style`を使用し、`display: none`からのスムーズなスプリング展開を実現します。
- `<d-hamburger target="#nav-id" breakpoint="md">`が`<d-navmenu id="nav-id">`を対象とする場合、モバイル画面ではデスクトップ用ナビゲーションが自動的に非表示になり、すべての直接リンクおよび`[d-submenu]`セクションが重複マークアップなしでモバイルドロワーのドリルダウンパネルへ変換されます。
- 完全なキーボードナビゲーション（トリガー間の`ArrowLeft` / `ArrowRight`、パネル内フォーカス移動の`ArrowDown` / `Enter`、閉じる`Escape`）をサポートします。
