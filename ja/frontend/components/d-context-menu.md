# d-context-menu

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/context-menu.mp4" type="video/mp4">
  </video>
</p>

境界の自動クランプと外部クリック抑制を備えた右クリックコンテキストメニューです。

```html
<div class="card p-2">
  right click inside this card
  <d-context-menu>
    <span>copy link</span>
    <span>duplicate item</span>
    <span>delete</span>
  </d-context-menu>
</div>
```

## 属性とプロパティ

| 属性     | プロパティ | 型        | デフォルト値 | フォーム | 説明                                         |
| :------- | :--------- | :-------- | :----------- | :------- | :------------------------------------------- |
| `opened` | `opened`   | `boolean` | `false`      | いいえ   | コンテキストメニューが開いているかどうか     |
| `parent` | `parent`   | `string`  | `""`         | いいえ   | 対象親要素のセレクター（デフォルトは親要素） |

## プログラム用api

```javascript
const menu = select('d-context-menu');

menu.open();
menu.open(pointerEvent);
menu.close();
menu.toggle();
dContextMenu.closeAll();

listen(menu, 'click', (e) => {
  const target = e.target;
  const action = target.closest('[data-action]')?.getAttribute('data-action');
  if (action) handleAction(action);
});
```

## イベント

| イベント | 詳細               | バブリング | 説明                                                       |
| :------- | :----------------- | :--------- | :--------------------------------------------------------- |
| `click`  | クリックされた要素 | はい       | 項目が選択されたときにコンポーネントエミッターを通じて送出 |
| `open`   | なし               | はい       | コンテキストメニューが開いたときに送出                     |
| `close`  | なし               | はい       | コンテキストメニューが閉じたときに送出                     |

## 動作仕様

- 親コンテナー内での右クリックで開きブラウザーのデフォルトコンテキストメニューを抑制します。
- メニュー寸法を計算しメニューが親や画面の端からはみ出さないよう座標を自動制限します。
- 外側をクリックすると下層要素のクリックアクションやリンクをトリガーすることなくメニューを即座に閉じます。
- 開く前に開いている他のすべてのコンテキストメニューを自動的に閉じます。
- javascriptで動的に生成する場合項目ラッパーは次のアニメーションフレームでのコンポーネントマウント時に作成されるためコンポーネント要素の`'click'`を直接リッスンします。
