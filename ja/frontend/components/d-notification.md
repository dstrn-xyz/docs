# d-notification

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/notification.mp4" type="video/mp4">
  </video>
</p>

リニアプログレスバーのカウントダウンと自動終了ライフサイクルを備えたトースト通知コンポーネントです。

```html
<d-notification timer="4000" opened>
  <p>project settings saved successfully</p>
</d-notification>
```

## 属性とプロパティ

| 属性     | プロパティ | 型        | デフォルト値 | フォーム | 説明                                            |
| :------- | :--------- | :-------- | :----------- | :------- | :---------------------------------------------- |
| `opened` | `opened`   | `boolean` | `false`      | いいえ   | トースト通知が表示されているかどうか            |
| `timer`  | `timer`    | `number`  | `0`          | いいえ   | ミリ秒単位の自動消去時間（`0`でタイマー無効化） |

## プログラム用api

```javascript
const toast = select('d-notification');

toast.show();
toast.hide();
await toast.destroy();
```

## イベント

| イベント | 詳細 | バブリング | 説明                   |
| :------- | :--- | :--------- | :--------------------- |
| `open`   | なし | はい       | 通知が開いたときに送出 |
| `close`  | なし | はい       | 通知が閉じたときに送出 |

## 動作仕様

- タイマー満了後に自動的に閉じるオプションのカウントダウンプログレスバー付きトースト通知を表示します。
