# d-sidebar

オフキャンバス開閉、アイコン縮小モード、モバイルドロワー対応のアプリケーションサイドバーです。

```html
<d-sidebar id="app-sidebar" collapsible="icon" variant="sidebar" persist="app_sidebar">
  <div slot="header">
    <span class="d-sidebar-label">ワークスペース</span>
  </div>

  <div d-sidebar-group>
    <div d-sidebar-group-label>ナビゲーション</div>
    <div d-sidebar-menu>
      <a d-sidebar-item href="/dashboard" active>
        <i class="dstrn-dashboard-line"></i>
        <span class="d-sidebar-label">ダッシュボード</span>
      </a>
      <div d-sidebar-item d-collapsible>
        <div d-sidebar-item-trigger>
          <i class="dstrn-folder-line"></i>
          <span class="d-sidebar-label">プロジェクト</span>
          <i class="dstrn-down-small-line d-sidebar-chevron"></i>
        </div>
        <div d-sidebar-sub>
          <a d-sidebar-sub-item href="/projects/active">アクティブ</a>
          <a d-sidebar-sub-item href="/projects/archive">アーカイブ</a>
        </div>
      </div>
    </div>
  </div>

  <div slot="footer">
    <span class="d-sidebar-label">ユーザー設定</span>
  </div>
  <div d-sidebar-rail></div>
</d-sidebar>
```

<a name="属性とプロパティd-sidebar"></a>
## 属性とプロパティ（d-sidebar）

| 属性 | プロパティ | 型 | デフォルト | フォーム | 説明 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `side` | `side` | string | `"left"` | いいえ | サイドバーの配置位置（`"left"`または`"right"`） |
| `collapsible` | `collapsible` | string | `"offcanvas"` | いいえ | 折りたたみ方式（`"offcanvas"`、`"icon"`、`"none"`） |
| `variant` | `variant` | string | `"sidebar"` | いいえ | 表示スタイル（`"sidebar"`、`"floating"`、`"inset"`） |
| `opened` | `opened` | boolean | `true` | いいえ | 開閉状態 |
| `breakpoint` | `breakpoint` | string | `"md"` | いいえ | モバイルドロワーに切り替えるブレークポイント（`"sm"`、`"md"`、`"lg"`、`"xl"`、メディアクエリ文字列、または`"none"`） |
| `shortcut` | `shortcut` | string | `"cmd+b"` | いいえ | 開閉をトグルするキーボードショートカット |
| `persist` | `persist` | string | `""` | いいえ | 開閉状態を保存するlocalstorageのキー |
| `contained` | `contained` | boolean | `false` | いいえ | ビューポートではなく親要素内にサイドバーを収める |
| `toggle-button` | `toggleButton` | boolean | `true` | いいえ | 端に常時表示される開閉トグルボタン（無効化する場合は`false`） |

<a name="プログラム用apid-sidebar"></a>
## プログラム用api（d-sidebar）

```javascript
const sidebar = select('d-sidebar');

sidebar.open();      // 開く（展開）
sidebar.close();     // 閉じる（折りたたみ）
sidebar.toggle();    // 開閉を切り替え

console.log(sidebar.isOpen);   // 展開中またはモバイル表示中の場合true
console.log(sidebar.isMobile); // ブレークポイント以下のモバイル幅の場合true
```

<a name="イベントd-sidebar"></a>
## イベント（d-sidebar）

| イベント | 詳細 | バブリング | 説明 |
| :--- | :--- | :--- | :--- |
| `open` | オブジェクト（`{ mobile }`） | はい | サイドバーが開いたときに送出 |
| `close` | オブジェクト（`{ mobile }`） | はい | サイドバーが閉じたときに送出 |

<a name="動作仕様d-sidebar"></a>
## 動作仕様（d-sidebar）

- 上部端に枠線をまたぐトグルボタン（dstrn-left-lineとdstrn-right-lineが切り替わる）が常時表示され、オフキャンバス折りたたみ時も画面端に収まり操作可能。
- アイコンモード（`collapsible="icon"`）時はラベルを隠して幅を縮小し、各アイコンに自動的にツールチップを表示。
- 折りたたみ可能サブメニューはcssの`grid-template-rows: 0fr to 1fr`を活用して滑らかに開閉。
- `[d-sidebar-rail]`のクリックや外部の`[d-sidebar-trigger]`要素（id指定は`[d-sidebar-trigger="#id"]`）から簡単にトグル可能。
- 設定されたブレークポイント以下では自動的にバックドロップ付きモバイルドロワーへ移行。
