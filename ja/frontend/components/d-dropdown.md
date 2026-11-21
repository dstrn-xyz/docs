# d-dropdown

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/dropdown.mp4" type="video/mp4">
  </video>
</p>

アコーディオン開閉コンテナーで、FAQなどに最適です。

```html
<d-dropdown header="account security">
  <div class="p-1 flex-column g-1">
    <d-checkbox text="two factor authentication"></d-checkbox>
    <a href="/sessions" class="btn fs-08">manage active sessions</a>
  </div>
</d-dropdown>
```

## 属性とプロパティ

| 属性     | プロパティ | 型        | デフォルト値 | フォーム | 説明                                                 |
| :------- | :--------- | :-------- | :----------- | :------- | :--------------------------------------------------- |
| `header` | `header`   | `string`  | `"dropdown"` | いいえ   | クリック可能なヘッダーバーに表示されるタイトル文字列 |
| `opened` | `opened`   | `boolean` | `false`      | いいえ   | ドロップダウンが展開されているかどうか               |

## プログラム用api

```javascript
const dropdown = select('d-dropdown');

dropdown.open();
dropdown.close();
dropdown.toggle();

console.log(dropdown.isOpened);
```

## イベント

| イベント | 詳細 | バブリング | 説明                                   |
| :------- | :--- | :--------- | :------------------------------------- |
| `open`   | なし | はい       | ドロップダウンパネルが開いたときに送出 |
| `close`  | なし | はい       | ドロップダウンパネルが閉じたときに送出 |

## 動作仕様

- ヘッダーをクリックするとシェブロンインジケーターのアニメーションとともに子コンテンツが展開または折りたたまれます。
- プロパティの変更時にヘッダー文字列と開閉状態が動的に更新されます。
