# d-drawer

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/drawer.mp4" type="video/mp4">
  </video>
</p>

アニメーション遷移とウェーブスタイリングを備えたオフキャンバスナビゲーションドロワーです。

```html
<d-drawer direction="left" id="main-drawer">
  <nav class="flex-column g-1 p-2">
    <a href="/dashboard">dashboard</a>
    <a href="/settings">settings</a>
  </nav>
</d-drawer>
```

## 属性とプロパティ

| 属性        | プロパティ  | 型        | デフォルト値 | フォーム | 説明                                                           |
| :---------- | :---------- | :-------- | :----------- | :------- | :------------------------------------------------------------- |
| `opened`    | `opened`    | `boolean` | `false`      | いいえ   | ドロワーが開いているかどうか                                   |
| `direction` | `direction` | `string`  | `"left"`     | いいえ   | スライドインする端（`"left"`、`"right"`、`"top"`、`"bottom"`） |

## プログラム用api

```javascript
const drawer = select('d-drawer');

await drawer.open();
await drawer.close();
await drawer.toggle();

console.log(drawer.isOpened, drawer.isAnimating);
```

## イベント

| イベント | 詳細 | バブリング | 説明                                 |
| :------- | :--- | :--------- | :----------------------------------- |
| `open`   | なし | はい       | オープンアニメーションの完了時に送出 |
| `close`  | なし | はい       | クローズアニメーションの完了時に送出 |

## 動作仕様

- 設定された端から複数レイヤーのウェーブアニメーションとともにスライド表示されます。
- プログラム用の`open()`、`close()`、`toggle()`メソッドはアニメーション完了時に解決するプロミスを返します。
