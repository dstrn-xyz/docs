# d-hamburger

プログラム可能なブレークポイントと多階層スライディングサブカテゴリーパネルを備えた、レスポンシブモバイルナビゲーションハンバーガートリガーおよびフルスクリーンナビゲーションオーバーレイです。

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <a d-link href="/" class="flex-row align-center">
      <img src="/img/logo.png" alt="logo">
    </a>

    <div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
      <a d-link href="/features" class="nav-link">features</a>
      <div d-submenu="framework">
        <a d-link href="/architecture">architecture</a>
        <a d-link href="/benchmarks">benchmarks</a>
      </div>
    </div>

    <d-hamburger target="#nav-links" breakpoint="md">
      <div slot="header">
        <img src="/img/logo.png" class="logo-mark">
      </div>
      <div slot="footer">
        <d-text-input placeholder="search docs"></d-text-input>
      </div>
    </d-hamburger>
  </div>
</nav>
```

## 属性とプロパティ

| 属性             | プロパティ       | 型        | デフォルト値      | フォーム | 説明                                                                                   |
| :--------------- | :--------------- | :-------- | :---------------- | :------- | :------------------------------------------------------------------------------------- |
| `opened`         | `opened`         | `boolean` | `false`           | いいえ   | オーバーレイが開いているかどうか                                                       |
| `breakpoint`     | `breakpoint`     | `string`  | `"md"`            | いいえ   | ブレークポイントトークン（`sm`、`md`、`lg`、`xl`、`xxl`、`ultra`）またはメディアクエリ |
| `target`         | `target`         | `string`  | `""`              | いいえ   | リンクを複製するデスクトップコンテナーのセレクター                                     |
| `animation`      | `animation`      | `string`  | `"slide-down"`    | いいえ   | オーバーレイ遷移アニメーション（`slide-down`、`fade`、`slide-right`、`slide-left`）    |
| `direction`      | `direction`      | `string`  | `"top"`           | いいえ   | 表示開始方向                                                                           |
| `size`           | `size`           | `string`  | `"1.5em"`         | いいえ   | em単位でのハンバーガーボタンサイズ                                                     |
| `color`          | `color`          | `string`  | `"currentColor"`  | いいえ   | ボタンの線色                                                                           |
| `activecolor`    | `activeColor`    | `string`  | `"var(--accent)"` | いいえ   | オープン時のボタン色                                                                   |
| `autohidetarget` | `autoHideTarget` | `boolean` | `true`            | いいえ   | ブレークポイント未満でデスクトップ対象コンテナーを自動的に非表示化                     |
| `title`          | `title`          | `string`  | `""`              | いいえ   | `slot="header"`省略時のフォールバックヘッダータイトル                                  |

## スロット

- `slot="header"`: モバイルオーバーレイ内のカスタムヘッダーコンテンツ
- `slot="footer"`: オーバーレイ下部のカスタムフッターコンテンツ

## プログラム用api

```javascript
const hamburger = select('d-hamburger');

await hamburger.open();
await hamburger.close();
await hamburger.toggle();

hamburger.navigateTo(subPanelElement, 'framework');
hamburger.navigateBack();
dHamburger.closeAll();
```

## イベント

| イベント | 詳細 | バブリング | 説明                           |
| :------- | :--- | :--------- | :----------------------------- |
| `open`   | なし | はい       | オーバーレイが開いたときに送出 |
| `close`  | なし | はい       | オーバーレイが閉じたときに送出 |

## 動作仕様

- 指定されたブレークポイント未満のときにモバイルボタンを自動表示しデスクトップナビゲーションを非表示にします。
- リンクコンテナー上の`d-submenu`属性を使用したネストされたスライディングサブパネルをサポートします。
- オープン中のページスクロールロックを自動管理します。
- リンクのクリック、Escapeキーの押下、またはページ間の遷移時に自動的に閉じます。
