# d-hold-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/hold-button.mp4" type="video/mp4">
  </video>
</p>

継続的なポインター長押しを要求することで破壊的処理の偶発的な実行を防ぐホールドボタンです。

```html
<d-hold-button delay="1500" class="btn btn-red" type="submit">
  hold to delete database
</d-hold-button>
```

## 属性とプロパティ

| 属性       | プロパティ | 型        | デフォルト値 | フォーム | 説明                                         |
| :--------- | :--------- | :-------- | :----------- | :------- | :------------------------------------------- |
| `delay`    | `delay`    | `number`  | `800`        | いいえ   | アクション実行前に必要な長押し時間（ミリ秒） |
| `type`     | `type`     | `string`  | `"button"`   | いいえ   | ボタンタイプ（`"button"`、`"submit"`）       |
| `disabled` | `disabled` | `boolean` | `false`      | いいえ   | ボタンが無効化されているかどうか             |

## プログラム用api

```javascript
const holdBtn = select('d-hold-button');

holdBtn.delay = 2000;
holdBtn.disabled = true;
```

## イベント

| イベント | 詳細                    | バブリング | 説明                               |
| :------- | :---------------------- | :--------- | :--------------------------------- |
| `click`  | ネイティブclickイベント | はい       | 全遅延時間の長押し完了後にのみ送出 |

## 動作仕様

- クリックアクションをトリガーする前に設定された時間だけ継続的な長押しを要求します。
- 完了前に離すとアクションをキャンセルし視覚的な進行状況をリセットします。
- `type="submit"`が設定されている場合は周囲のフォームを自動送信します。
