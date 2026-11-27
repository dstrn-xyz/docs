# メール

- [メール](#メール)
  - [はじめに](#はじめに)
  - [設定](#設定)
  - [メールの送信](#メールの送信)
    - [ビューテンプレートのコンパイル](#ビューテンプレートのコンパイル)
    - [プレーンテキストメール](#プレーンテキストメール)
    - [メーラーの切り替え](#メーラーの切り替え)
  - [メールのキュー送信](#メールのキュー送信)
  - [メールのテスト](#メールのテスト)

<a name="はじめに"></a>

## はじめに

dframeworkはローカルログ出力、ゼロ依存のsmtp配信、aws ses、およびカスタムwebhookをサポートするメール転送サービスを標準で備えています。

<a name="設定"></a>

## 設定

メール設定は`config/mail.js`で定義します。

```javascript
import { Env } from 'dframework';

export default {
  default: Env.value('MAIL_MAILER', 'log'),
  mailers: {
    log: {
      transport: 'log'
    },
    smtp: {
      transport: 'smtp',
      host: Env.value('MAIL_HOST', '127.0.0.1'),
      port: Env.value('MAIL_PORT', 587),
      encryption: Env.value('MAIL_ENCRYPTION', 'tls'),
      username: Env.value('MAIL_USERNAME'),
      password: Env.value('MAIL_PASSWORD')
    },
    ses: {
      transport: 'ses',
      region: Env.value('AWS_DEFAULT_REGION', 'us-east-1'),
      key: Env.value('AWS_ACCESS_KEY_ID'),
      secret: Env.value('AWS_SECRET_ACCESS_KEY'),
      endpoint: Env.value('MAIL_ENDPOINT')
    },
    webhook: {
      transport: 'webhook',
      url: Env.value('MAIL_WEBHOOK_URL')
    }
  },
  from: {
    address: Env.value('MAIL_FROM_ADDRESS', 'hello@example.com'),
    name: Env.value('MAIL_FROM_NAME', 'dframework')
  }
};
```

<a name="メールの送信"></a>

## メールの送信

グローバルな`Mail`ファサードは`Mail.to()`を通じた流暢なビルダーインターフェースと、`Mail.send()`による直接送信を提供します。

```javascript
// メッセージオブジェクトによる直接送信
await Mail.send({
  to: 'user@example.com',
  subject: 'welcome',
  text: 'welcome to dframework'
});
```

<a name="ビューテンプレートのコンパイル"></a>

### ビューテンプレートのコンパイル

```javascript
await Mail.to('user@example.com')
  .subject('dframeworkへようこそ')
  .send('emails.welcome', { user });
```

メールテンプレートをレンダリングする際、ブラウザ用のアセットタグ（`dstrn.css`および`dstrn.js`）を挿入せずにクリーンに`.d`ビューがコンパイルされます。

<a name="プレーンテキストメール"></a>

### プレーンテキストメール

```javascript
await Mail.to('user@example.com')
  .subject('セキュリティ通知')
  .raw('新しいデバイスからのログインが記録されました');
```

<a name="メーラーの切り替え"></a>

### メーラーの切り替え

`config/mail.js`で設定された特定のメーラーを使用して送信する場合：

```javascript
await Mail.mailer('ses')
  .to('user@example.com')
  .subject('welcome')
  .send('emails.welcome', { user });
```

<a name="メールのキュー送信"></a>

## メールのキュー送信

バックグラウンドで非同期にメールを送信するには`queue()`を使用します。

```javascript
await Mail.to('user@example.com')
  .subject('注文確認')
  .queue('emails.order', { order });
```

これによりhttpレスポンスを遅延させることなく、アクティブなキューのドライバー上で`Job.dispatch`経由で配信されます。

<a name="メールのテスト"></a>

## メールのテスト

テスト環境では`Mail.fake()`を使用してメール送信をインターセプトします。

```javascript
Mail.fake();

await Mail.to('user@example.com')
  .subject('welcome')
  .send('emails.welcome', { user });

Mail.assertSent('welcome');
Mail.assertSent(m => m.to.some(r => (typeof r === 'string' ? r : r.address) === 'user@example.com'));
Mail.assertSentCount(1);
```

キューに投入されたメールを検証する場合：

```javascript
Mail.assertQueued('注文確認');
```

操作中にメールが一切送信またはキュー投入されなかったことを検証する場合：

```javascript
Mail.assertNothingSent();
```

`Tester`を使用した統合テストでは、`Tester`ファサード上で直接ヘルパーメソッドを利用できます。

```javascript
Tester.fakeMail();

await Tester.post('/register', { email: 'user@example.com' });

Tester.assertSentMail('welcome');
Tester.assertSentMailCount(1);
Tester.assertQueuedMail('verify email');
```

操作中にメールが一切送信またはキュー投入されなかったことをアサートする場合：

```javascript
Tester.assertNothingMailSent();
```

`assertSent`や`assertQueued`に渡す述語関数は`{ to, cc, bcc, from, subject, view, template, data, replyTo, html, text }`を持つオブジェクトを受け取ります。文字列を渡すとテンプレートビュー名または件名と一致するか検証します。`to`、`cc`、`bcc`内の宛先はメールアドレス文字列または`{ address, name }`オブジェクトとして格納されます。
