# d-menubar

トップレベルトリガー、アクティブモード時の高速ホバー切り替え、ネストされたサブメニュー、チェック項目、ラジオグループ、統合キーボードショートカットを備えたデスクトップスタイルのメニューバーです。

```html
<d-menubar>
  <div d-menu="file">
    <span slot="trigger">file</span>
    <div slot="content">
      <div d-item value="new-tab" shortcut="cmd+t">new tab</div>
      <div d-item value="new-window" shortcut="cmd+n">new window</div>
      <div d-item value="incognito" disabled>new incognito window</div>
      <hr>
      <div d-submenu="share">
        <span slot="trigger">share</span>
        <div slot="content">
          <div d-item value="email-link">email link</div>
          <div d-item value="copy-link" shortcut="cmd+c">copy link</div>
        </div>
      </div>
      <hr>
      <div d-item value="print" shortcut="cmd+p">print</div>
    </div>
  </div>
  <div d-menu="edit">
    <span slot="trigger">edit</span>
    <div slot="content">
      <div d-item value="undo" shortcut="cmd+z">undo</div>
      <div d-item value="redo" shortcut="cmd+shift+z">redo</div>
    </div>
  </div>
  <div d-menu="view">
    <span slot="trigger">view</span>
    <div slot="content">
      <div d-check value="show-bookmarks" checked shortcut="cmd+b">show bookmarks</div>
      <div d-check value="show-full-urls">show full urls</div>
    </div>
  </div>
  <div d-menu="profiles">
    <span slot="trigger">profiles</span>
    <div slot="content">
      <div d-radio-group name="profile" value="work">
        <div d-radio value="personal">personal</div>
        <div d-radio value="work">work</div>
        <div d-radio value="guest">guest</div>
      </div>
    </div>
  </div>
</d-menubar>
```

<a name="属性とプロパティd-menubar"></a>
## 属性とプロパティ（d-menubar）

| 属性       | プロパティ | 型        | デフォルト値 | フォーム | 説明                                                                        |
| :--------- | :--------- | :-------- | :----------- | :------- | :-------------------------------------------------------------------------- |
| `items`    | `items`    | `array`   | `null`       | いいえ   | 構造化メニュー定義配列（宣言的子マークアップの代替）                        |
| `disabled` | `disabled` | `boolean` | `false`      | いいえ   | メニューバーの操作を無効化                                                  |

宣言的マークアップ属性：

- `d-menu="<id>"`: トップレベルメニューコンテナー（`<d-menubar>`の子要素）
- `slot="trigger"`: `[d-menu]`または`[d-submenu]`内のトリガー要素
- `slot="content"`: アイテム群を含むドロップダウンパネルラッパー
- `d-item`: アクション項目（`value`属性で識別子、`shortcut`で`<d-key>`バッジ、`disabled`で無効化）
- `d-check`: 切り替え可能なチェック項目（`value`, `checked`, `shortcut`, `disabled`）
- `d-radio-group`: 相互排他グループ（`name`および`value`属性）、内部に`[d-radio]`項目を配置
- `d-submenu="<id>"`: ネストされたサブメニューコンテナー（`slot="trigger"`と`slot="content"`を含む）
- `<hr>`（または`.d-menubar-sep`）: 境界線区切り

<a name="プログラム用apid-menubar"></a>
## プログラム用api（d-menubar）

```javascript
const bar = select('d-menubar');

bar.openMenu(0);
bar.closeAllMenus();
bar.items = [
  {
    id: 'file',
    title: 'file',
    items: [
      { value: 'new', label: 'new tab', shortcut: 'cmd+t' },
      'separator',
      { value: 'save', label: 'save', shortcut: 'cmd+s' }
    ]
  }
];

// すべてのメニューバーインスタンスを閉じる
dMenubar.closeAll();
```

<a name="イベントd-menubar"></a>
## イベント（d-menubar）

| イベント | 詳細                           | バブリング | 説明                                           |
| :------- | :----------------------------- | :--------- | :--------------------------------------------- |
| `select` | `{ value, item, menu }`        | はい       | アクション項目が実行されたときに送出           |
| `change` | `{ value, checked }` または `{ name, value }` | はい | チェック項目またはラジオ選択が変更されたときに送出 |
| `open`   | `{ menu, index }`              | はい       | トップレベルメニューパネルが開いたときに送出   |
| `close`  | `{ menu, index }`              | はい       | メニューパネルが閉じたときに送出               |

<a name="動作仕様d-menubar"></a>
## 動作仕様（d-menubar）

- `d-popover`を介してブラウザのトップレイヤーにドロップダウンパネルをレンダリングし、ネストされたオーバーフローコンテナー内でのクリッピングを回避します。
- トップレベルメニューをクリックするとパネルが開き、メニューモードが有効化されます。その後はポインターを隣接トリガーに移動するだけで瞬時にメニューが切り替わります。
- サブメニューはホバー時に猶予遅延を伴って展開し、ネスト階層をサポートします。
- 完全なキーボードナビゲーション（トップトリガー間の`ArrowLeft` / `ArrowRight`、メニュー項目間の`ArrowDown` / `ArrowUp`、サブメニュー内外への`ArrowRight` / `ArrowLeft`、実行のための`Enter` / `Space`、閉じるための`Escape`）をサポートします。
