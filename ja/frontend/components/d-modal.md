# d-modal

ページスクロールロックとバックドロップによる意図しない終了の防止機能を備えたモーダルダイアログコンテナーです。

```html
<d-modal id="confirm-modal" persistent>
  <div class="flex-column g-1">
    <h3>confirm deletion</h3>
    <p class="text-content-l">this action cannot be undone.</p>
    <div class="flex-row justify-end g-1 mt-1">
      <button class="btn fs-08" onclick="this.closest('d-modal').close()">cancel</button>
      <button class="btn btn-red fs-08">delete</button>
    </div>
  </div>
</d-modal>
```

## 属性とプロパティ

| 属性         | プロパティ   | 型        | デフォルト値 | フォーム | 説明                                                        |
| :----------- | :----------- | :-------- | :----------- | :------- | :---------------------------------------------------------- |
| `opened`     | `opened`     | `boolean` | `false`      | いいえ   | モーダルが現在表示されているかどうか                        |
| `persistent` | `persistent` | `boolean` | `false`      | いいえ   | バックドロップクリックで閉じた際に要素をDOM内に残すかどうか |

## プログラム用api

```javascript
const modal = select('d-modal');

modal.open();
modal.close();
await modal.destroy();
```

## イベント

| イベント | 詳細 | バブリング | 説明                       |
| :------- | :--- | :--------- | :------------------------- |
| `open`   | なし | はい       | モーダルが開いたときに送出 |
| `close`  | なし | はい       | モーダルが閉じたときに送出 |

## 動作仕様とスタイリング境界

- オーバーレイダイアログを表示し表示中はページスクロールを防止します。
- `persistent`が省略されている（または`false`の）場合、バックドロップをクリックすると要素が閉じられ破棄され（`destroy()`）、DOMから削除されます。
- `persistent`が指定されている（`<d-modal persistent>`）場合、バックドロップをクリックすると要素をDOMから削除せずにダイアログを閉じ（`close()`）、再度開くことができます。
- `<d-modal>`は`.d-modal-wrapper`（`var(--container-l)`背景、ボーダー、角丸、シャドウを提供）および`.d-modal-content`（スクロール管理付きフレックスコンテナー）を自動注入します。
- モーダルの外観をカスタマイズまたは再スタイルするにはCSSで`d-modal .d-modal-wrapper`または`d-modal .d-modal-content`を対象にします。
