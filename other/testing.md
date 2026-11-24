# testing

- [testing](#testing)
  - [introduction](#introduction)
  - [running tests](#running-tests)
  - [generating tests](#generating-tests)
  - [http integration tests](#http-integration-tests)
    - [making requests](#making-requests)
    - [named route dispatch](#named-routes)
    - [direct controller execution](#direct-controller-execution)
    - [acting as a user](#acting-as-a-user)
    - [available assertions](#available-assertions)
  - [service fakes](#service-fakes)
    - [faking background jobs](#faking-background-jobs)
    - [faking websocket events](#faking-websocket-events)
  - [database assertions](#database-assertions)
    - [migrations, seeders, and transactions](#migrations-and-transactions)
  - [browser and page interaction tests](#browser-and-page-interaction-tests)
    - [navigating pages](#navigating-pages)
    - [form interactions and clicks](#form-interactions-and-clicks)
    - [browser assertions](#browser-assertions)

<a name="introduction"></a>

## introduction

dframework includes a built in test harness with zero external dependencies, running directly on node's native test runner (`node:test`). applications can perform fast in process http requests without spinning up network sockets, test database state, and simulate browser interactions with full form and navigation fidelity in continuous integration pipelines.

the `dframework/testing` module exports top level runner and assertion primitives (`test`, `it`, `describe`, `Tester`, and strict `assert`). test files use the unified `Tester` facade (`Tester.get`, `Tester.post`, `Tester.route`, `Tester.actingAs`, `Tester.fake`, `Tester.assertDatabaseHas`) with zero wrapper boilerplate. like other facades, `Tester` is also available globally on globalThis in the test environment. the runner supports modifiers: `test.only()` to isolate focus, `test.skip()` to temporarily ignore cases, `test.todo()` to define pending specifications with or without implementation callbacks, and `test.transaction()` to automatically isolate database changes.

<a name="running-tests"></a>

## running tests

you can run the entire test suite or target specific files using `dstrn test`:

```bash
# run all tests in tests/
dstrn test

# run a specific test file or directory
dstrn test tests/feature/UserTest.test.js
dstrn test tests/unit

# watch mode for instant re running on file changes
dstrn test --watch

# run only tests marked with { only: true }
dstrn test --only
```

`dstrn test` automatically sets `APP_ENV=testing` and enables experimental sqlite support when running under node 22 or 23. tests strictly require a `.env.testing` file in the project root; the framework does not fall back to `.env`, preventing accidental data mutations in your development or production databases.

<a name="generating-tests"></a>

## generating tests

you can scaffold new test files using `dstrn make:test`:

```bash
# create a feature test in tests/feature/
dstrn make:test UserTest
dstrn make:test UserTest --feature

# create a browser test in tests/browser/
dstrn make:test CheckoutTest --browser

# create a unit test in tests/unit/
dstrn make:test CalculatorTest --unit

# nested directories are created automatically
dstrn make:test Auth/RegistrationTest
```

<a name="http-integration-tests"></a>

## http integration tests

the `Tester` facade provides an in process http dispatcher that passes synthetic request streams directly through the framework pipeline. this guarantees zero mock drift while avoiding live socket binding overhead.

<a name="making-requests"></a>

### making requests

```javascript
import { test, Tester } from 'dframework/testing';

test('returns active users', async () => {
  const res = await Tester.get('/api/users');

  res.assertOk();
  res.assertJson({ success: true });
});
```

supported request methods include `get`, `post`, `put`, `patch`, `delete`, `head`, `options`, and custom verbs via `Tester.request(method, uri, data, headers)`. cookies and session state are preserved across sequential requests within the same client instance.

you can configure custom headers, cookies, or disable automated csrf token generation:

```javascript
// attach custom headers or cookies
Tester.withHeaders({ 'x-test-header': 'custom' });
Tester.withCookies({ theme: 'dark' });

// disable automatic csrf token generation for mutating requests
Tester.withoutCsrf();
```

<a name="named-routes"></a>

### named route dispatch

dispatch requests using defined route names and parameter maps:

```javascript
test('named route order lookup', async () => {
  const res = await Tester.route('orders.show', { id: 42 });

  res.assertOk();
  res.assertJson({ orderId: '42' });
});
```

<a name="direct-controller-execution"></a>

### direct controller execution

test controller actions or middleware chains directly using `Tester.call()`:

```javascript
test('direct controller action', async () => {
  const res = await Tester.call('UserController@show', {
    uri: '/users/:id',
    params: { id: 1 }
  });

  res.assertOk();
});
```

<a name="acting-as-a-user"></a>

### acting as a user

use `actingAs` to authenticate a user before executing requests:

```javascript
test('authenticated profile view', async () => {
  Tester.actingAs({ id: 1, email: 'alice@example.com' });

  const res = await Tester.get('/profile');
  res.assertOk();
  res.assertSee('alice@example.com');
});
```

<a name="available-assertions"></a>

### available assertions

the response object returned by synthetic requests provides chainable assertion helpers:

| assertion                 | description                                                        |
| ------------------------- | ------------------------------------------------------------------ |
| `assertStatus(code)`      | assert the response status code                                    |
| `assertOk()`              | assert status 200                                                  |
| `assertCreated()`         | assert status 201                                                  |
| `assertNotFound()`        | assert status 404                                                  |
| `assertForbidden()`       | assert status 403                                                  |
| `assertUnauthorized()`    | assert status 401                                                  |
| `assertRedirect(uri)`     | assert a redirect status and matching location                     |
| `assertJson(expected)`    | assert the response json contains the expected structure or values |
| `assertJsonMissing(key)`  | assert a key is absent from the response json                      |
| `assertSee(text)`         | assert response contains the given substring                       |
| `assertDontSee(text)`     | assert response does not contain the given substring               |
| `assertHeader(name, val)` | assert response header value                                       |

<a name="service-fakes"></a>

## service fakes

dframework provides test doubles and spies for background jobs and websocket events via `Tester.fake({ jobs: true, socket: true })` (or `Tester.fakeJobs()` / `Tester.fakeSocket()`, or passing service names like `Tester.fake('jobs')` or `Tester.fake(['jobs', 'socket'])`). fakes intercept dispatches in memory, prevent real execution, provide expressive assertions, and automatically restore state when the test completes.

<a name="faking-background-jobs"></a>

### faking background jobs

calling `Tester.fake({ jobs: true })` (or `Tester.fakeJobs()`) intercepts calls to `Job.dispatch()` and records dispatched jobs for inspection:

```javascript
import { test, Tester } from 'dframework/testing';

test('dispatches confirmation email on checkout', async () => {
  Tester.fake({ jobs: true });

  const res = await Tester.post('/checkout', { cartId: 10 });
  res.assertOk();

  Tester.assertDispatched('SendOrderConfirmationJob');
  Tester.assertDispatched('SendOrderConfirmationJob', (payload) => payload.cartId === 10);
  Tester.assertDispatchedTimes('SendOrderConfirmationJob', 1);
  Tester.assertNotDispatched('CancelOrderJob');
});
```

| assertion | description |
| --- | --- |
| `Tester.assertDispatched(job, callback)` | assert job was dispatched, optionally matching payload predicate |
| `Tester.assertNotDispatched(job, callback)` | assert job was not dispatched |
| `Tester.assertDispatchedTimes(job, count)` | assert job was dispatched an exact number of times |
| `Tester.assertNothingDispatched()` | assert no background jobs were dispatched |
| `Tester.dispatched(job, callback)` | retrieve dispatched job instances matching filter |

<a name="faking-websocket-events"></a>

### faking websocket events

calling `Tester.fake({ socket: true })` (or `Tester.fakeSocket()`) intercepts real time websocket broadcasts and direct user messages without requiring a running websocket server:

```javascript
import { test, Tester } from 'dframework/testing';

test('broadcasts message and notifies recipient', async () => {
  Tester.fake({ socket: true });

  await Tester.post('/messages', { toUser: 42, text: 'hello' });

  Tester.assertBroadcast('message.sent');
  Tester.assertSentToUser(42, 'notification.new');
  Tester.assertNotSentToUser(99, 'notification.new');
});
```

| assertion | description |
| --- | --- |
| `Tester.assertBroadcast(event, callback)` | assert event was broadcast to all clients |
| `Tester.assertNotBroadcast(event, callback)` | assert event was not broadcast |
| `Tester.assertSentToUser(userOrId, event, callback)` | assert event was sent to specific user id |
| `Tester.assertNotSentToUser(userOrId, event, callback)` | assert event was not sent to user |
| `Tester.assertSentToSession(sessionId, event, callback)` | assert event was sent to specific session |
| `Tester.assertSent(event, callback)` | assert event was sent across any channel |
| `Tester.assertNotSent(event, callback)` | assert event was not sent across any channel |
| `Tester.assertSentTimes(event, count)` | assert event was sent an exact number of times |
| `Tester.assertNothingSent()` | assert no socket events were emitted |
| `Tester.assertRevoked(userOrId, guardName)` | assert user websocket session was revoked |
| `Tester.sent(event, callback)` | retrieve sent socket events matching filter |
| `Tester.broadcasted(event, callback)` | retrieve broadcasted socket events matching filter |

<a name="database-assertions"></a>

## database assertions

the `Tester` facade exposes asynchronous database assertion helpers:

```javascript
import { test, Tester } from 'dframework/testing';

Tester.useDatabaseTransactions();

test('user registration records to database', async () => {
  await Tester.post('/register', {
    name: 'bob',
    email: 'bob@example.com'
  });

  await Tester.assertDatabaseHas('users', { email: 'bob@example.com' });
  await Tester.assertDatabaseMissing('users', { email: 'charlie@example.com' });
  await Tester.assertDatabaseCount('users', 1);
});
```

<a name="migrations-and-transactions"></a>

### migrations, seeders, and transactions

manage database state with dedicated helpers and automatically isolate test mutations using transactions:

```javascript
test('database transactions and seeding', async () => {
  await Tester.migrate();
  await Tester.rollback();
  await Tester.migrate();
  await Tester.seed('UserSeeder');

  await Tester.useTransaction(async () => {
    await Tester.post('/users', { name: 'Alice' });
    await Tester.assertDatabaseHas('users', { name: 'Alice' });
  });

  await Tester.assertDatabaseMissing('users', { name: 'Alice' });
});
```

<a name="browser-and-page-interaction-tests"></a>

## browser and page interaction tests

the `browse()` client runs a lightweight simulated dom parser that executes form serialization, link traversal, input typing, checkboxes, and select dropdowns without requiring heavy headless browser binaries in continuous integration. note that simulated dom evaluates server rendered html using whitespace separated descendant selectors and `:not()` without running a client javascript runtime or complex combinators (such as `>`, `+`, or `~`).

<a name="navigating-pages"></a>

### navigating pages

```javascript
import { test, browse } from 'dframework/testing';

test('visits and navigates via link', async () => {
  const page = await browse('/');

  page.assertSee('welcome to dframework');
  page.assertElement('nav');

  await page.click('#docs-link');
  page.assertPath('/docs');
});
```

<a name="form-interactions-and-clicks"></a>

### form interactions and clicks

```javascript
test('fill and submit contact form', async () => {
  const page = await browse('/contact');

  page.type('input[name="email"]', 'visitor@example.com');
  page.type('textarea[name="message"]', 'hello framework');
  page.check('input[name="newsletter"]');
  page.select('select[name="topic"]', 'support');

  await page.click('button[type="submit"]');

  page.assertPath('/contact/success');
  page.assertSee('thank you for reaching out');
});
```

<a name="browser-assertions"></a>

### browser assertions

| assertion                      | description                                         |
| ------------------------------ | --------------------------------------------------- |
| `assertPath(path)`             | assert current url pathname                         |
| `assertSee(text)`              | assert rendered text contains substring             |
| `assertDontSee(text)`          | assert rendered text does not contain substring     |
| `assertElement(selector)`      | assert element matching css selector exists         |
| `assertMissing(selector)`      | assert element matching css selector does not exist |
| `assertValue(selector, value)` | assert input or select element has expected value   |
| `assertChecked(selector)`      | assert checkbox is checked                          |
| `assertNotChecked(selector)`   | assert checkbox is unchecked                        |