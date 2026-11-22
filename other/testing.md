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
  - [database assertions](#database-assertions)
    - [migrations, seeders, and transactions](#migrations-and-transactions)
  - [browser and page interaction tests](#browser-and-page-interaction-tests)
    - [navigating pages](#navigating-pages)
    - [form interactions and clicks](#form-interactions-and-clicks)
    - [browser assertions](#browser-assertions)

<a name="introduction"></a>

## introduction

dframework includes a built in test harness with zero external dependencies, running directly on node's native test runner (`node:test`). applications can perform fast in process http requests without spinning up network sockets, test database state, and simulate browser interactions with full form and navigation fidelity in continuous integration pipelines.

the `dframework/testing` module exports all test runner and assertion primitives (`test`, `describe`, `it`, `before`, `after`, `beforeEach`, `afterEach`, and strict `assert`). application test files only need to import from `dframework/testing` without requiring any direct `node:test` or `node:assert` imports.

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

the `testApp()` client provides an in process http dispatcher that passes synthetic request streams directly through the framework pipeline. this guarantees zero mock drift while avoiding live socket binding overhead.

<a name="making-requests"></a>

### making requests

```javascript
import { describe, it, assert, testApp } from 'dframework/testing';

describe('users api', () => {
  it('returns active users', async () => {
    const app = await testApp();
    const res = await app.get('/api/users');

    res.assertOk();
    res.assertJson({ success: true });
  });
});
```

supported request methods include `get`, `post`, `put`, `patch`, `delete`, and `head`. cookies and session state are preserved across sequential requests within the same client instance.

you can configure custom headers, cookies, or disable automated csrf token generation:

```javascript
const app = await testApp();

// attach custom headers or cookies
app.withHeaders({ 'x-test-header': 'custom' });
app.withCookies({ theme: 'dark' });

// disable automatic csrf token generation for mutating requests
app.withoutCsrf();
```

<a name="named-routes"></a>

### named route dispatch

dispatch requests using defined route names and parameter maps:

```javascript
test('named route order lookup', async () => {
  const app = await testApp();
  const res = await app.route('orders.show', { id: 42 });

  res.assertOk();
  res.assertJson({ orderId: '42' });
});
```

<a name="direct-controller-execution"></a>

### direct controller execution

test controller actions or middleware chains directly using `app.call()`:

```javascript
test('direct controller action', async () => {
  const app = await testApp();
  const res = await app.call('UserController@show', {
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
  const app = await testApp();
  app.actingAs({ id: 1, email: 'alice@example.com' });

  const res = await app.get('/profile');
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

<a name="database-assertions"></a>

## database assertions

the test harness exposes database assertion helpers directly on the test app instance:

```javascript
test('user registration records to database', async () => {
  const app = await testApp();

  await app.post('/register', {
    name: 'Bob',
    email: 'bob@example.com'
  });

  await app.assertDatabaseHas('users', { email: 'bob@example.com' });
  await app.assertDatabaseMissing('users', { email: 'charlie@example.com' });
  await app.assertDatabaseCount('users', 1);
});
```

<a name="migrations-and-transactions"></a>

### migrations, seeders, and transactions

manage database state with dedicated helpers and automatically isolate test mutations using transactions:

```javascript
test('database transactions and seeding', async () => {
  const app = await testApp();
  await app.migrate();
  await app.rollback();
  await app.migrate();
  await app.seed('UserSeeder');

  await app.useTransaction(async () => {
    await app.post('/users', { name: 'Alice' });
    await app.assertDatabaseHas('users', { name: 'Alice' });
  });

  await app.assertDatabaseMissing('users', { name: 'Alice' });
});
```

<a name="browser-and-page-interaction-tests"></a>

## browser and page interaction tests

the `browse()` client runs a lightweight simulated dom parser that executes form serialization, link traversal, input typing, checkboxes, and select dropdowns without requiring heavy headless browser binaries in continuous integration. note that simulated dom evaluates server rendered html using whitespace separated descendant selectors and `:not()` without running a client javascript runtime or complex combinators (such as `>`, `+`, or `~`).

<a name="navigating-pages"></a>

### navigating pages

```javascript
import { describe, it, assert, browse } from 'dframework/testing';

describe('homepage', () => {
  it('visits and navigates via link', async () => {
    const page = await browse('/');

    page.assertSee('welcome to dframework');
    page.assertElement('nav');

    await page.click('#docs-link');
    page.assertPath('/docs');
  });
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