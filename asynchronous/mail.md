# mail

- [mail](#mail)
  - [introduction](#introduction)
  - [configuration](#configuration)
  - [sending mail](#sending-mail)
    - [compiling view templates](#compiling-view-templates)
    - [plain text mail](#plain-text-mail)
    - [switching mailers](#switching-mailers)
  - [queueing mail](#queueing-mail)
  - [testing mail](#testing-mail)

<a name="introduction"></a>

## introduction

dframework includes built in mail transport services supporting local log output, zero dependency smtp delivery, aws ses, and custom webhooks.

<a name="configuration"></a>

## configuration

mail configuration is defined in `config/mail.js`.

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

<a name="sending-mail"></a>

## sending mail

the global `Mail` facade provides a fluent builder interface via `Mail.to()` as well as direct delivery via `Mail.send()`:

```javascript
// direct send with message object
await Mail.send({
  to: 'user@example.com',
  subject: 'welcome',
  text: 'welcome to dframework'
});
```

<a name="compiling-view-templates"></a>

### compiling view templates

```javascript
await Mail.to('user@example.com')
  .subject('welcome to dframework')
  .send('emails.welcome', { user });
```

when rendering email templates, `.d` views are compiled cleanly without browser asset tags (`dstrn.css` and `dstrn.js`).

<a name="plain-text-mail"></a>

### plain text mail

```javascript
await Mail.to('user@example.com')
  .subject('security alert')
  .raw('a login from a new device was recorded');
```

<a name="switching-mailers"></a>

### switching mailers

to send using a specific mailer configured in `config/mail.js`:

```javascript
await Mail.mailer('ses')
  .to('user@example.com')
  .subject('welcome')
  .send('emails.welcome', { user });
```

<a name="queueing-mail"></a>

## queueing mail

to send mail asynchronously in the background, use `queue()`:

```javascript
await Mail.to('user@example.com')
  .subject('order confirmation')
  .queue('emails.order', { order });
```

this delegates delivery to `Job.dispatch` on your active queue driver without delaying the http response.

<a name="testing-mail"></a>

## testing mail

use `Mail.fake()` to intercept mail in test environments:

```javascript
Mail.fake();

await Mail.to('user@example.com')
  .subject('welcome')
  .send('emails.welcome', { user });

Mail.assertSent('welcome');
Mail.assertSent(m => m.to.some(r => (typeof r === 'string' ? r : r.address) === 'user@example.com'));
Mail.assertSentCount(1);
```

you can also assert queued mail:

```javascript
Mail.assertQueued('order confirmation');
```

to assert that no mail was sent or queued during an operation:

```javascript
Mail.assertNothingSent();
```

when writing integration tests using `Tester`, helper methods are available directly on the `Tester` facade:

```javascript
Tester.fakeMail();

await Tester.post('/register', { email: 'user@example.com' });

Tester.assertSentMail('welcome');
Tester.assertSentMailCount(1);
Tester.assertQueuedMail('verify email');
```

to assert that no mail was dispatched or queued during an operation:

```javascript
Tester.assertNothingMailSent();
```

predicates passed to `assertSent` or `assertQueued` receive an object with `{ to, cc, bcc, from, subject, view, template, data, replyTo, html, text }`. passing a string matches against either the template view name or the subject line. recipients in `to`, `cc`, and `bcc` are normalized address entries which can be email strings or `{ address, name }` objects.
