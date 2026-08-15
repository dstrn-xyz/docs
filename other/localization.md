# localization

- [localization](#localization)
  - [introduction](#introduction)
  - [defining translations](#defining-translations)
    - [directory structure](#directory-structure)
    - [translation files](#translation-files)
    - [nested keys](#nested-keys)
  - [retrieving translations](#retrieving-translations)
    - [the t helper](#the-t-helper)
    - [in views](#in-views)
    - [fallback behavior](#fallback-behavior)
    - [variable interpolation](#variable-interpolation)
    - [japanese furigana (ruby) support](#japanese-furigana-ruby-support)
  - [locale detection](#locale-detection)
    - [detection order](#detection-order)
    - [default locale](#default-locale)
  - [setting the locale](#setting-the-locale)
    - [persistent locale](#persistent-locale)
    - [request-only locale](#request-only-locale)
    - [reading the locale](#reading-the-locale)
  - [caching and hot reload](#caching-and-hot-reload)

<a name="introduction"></a>

## introduction

dframework provides a lightweight localization system for building multilingual applications. translations are stored as json files organized by locale, and the framework provides helpers for retrieving translated strings in both controllers and views. locale detection happens automatically from cookies and the `Accept-Language` header, and the active locale can be changed per request or persisted to the user's session.

<a name="defining-translations"></a>

## defining translations

<a name="directory-structure"></a>

### directory structure

translation files are stored in the `lang` directory at the root of your project. each locale has its own subdirectory named with the locale code. inside each locale directory, you create json files that group related translation keys.

```
lang/
├── en/
│   ├── common.json
│   ├── auth.json
│   └── dashboard.json
├── ja/
│   ├── common.json
│   ├── auth.json
│   └── dashboard.json
└── es/
    ├── common.json
    └── auth.json
```

<a name="translation-files"></a>

### translation files

each translation file is a standard json object mapping keys to translated strings.

```json
{
  "welcome": "welcome to dframework",
  "goodbye": "see you next time",
  "greeting": "hello, welcome back"
}
```

```json
{
  "welcome": "dframeworkへようこそ",
  "goodbye": "またお会いしましょう",
  "greeting": "こんにちは、おかえりなさい"
}
```

<a name="nested-keys"></a>

### nested keys

translation files support nested objects. you access nested keys using dot notation.

```json
{
  "buttons": {
    "save": "save changes",
    "cancel": "cancel",
    "delete": "delete permanently"
  },
  "messages": {
    "success": "operation completed",
    "error": "something went wrong"
  }
}
```

```javascript
await t(req, 'common.buttons.save');
// "save changes"
```

<a name="retrieving-translations"></a>

## retrieving translations

<a name="the-t-helper"></a>

### the t helper

the `t()` function is the primary way to retrieve translations. it is available globally. it accepts the current request object, a dot notation key, and an optional fallback value.

```javascript
const welcome = await t(req, 'common.welcome');
const missing = await t(req, 'common.nonexistent', 'default text');
```

the first segment of the key is the filename (without the `.json` extension), and the remaining segments form the path within the json object.

| key                     | file                           | path             |
| ----------------------- | ------------------------------ | ---------------- |
| `common.welcome`        | `lang/{locale}/common.json`    | `welcome`        |
| `auth.errors.invalid`   | `lang/{locale}/auth.json`      | `errors.invalid` |
| `dashboard.stats.users` | `lang/{locale}/dashboard.json` | `stats.users`    |

the function is asynchronous because it reads translation files from disk on first access.

<a name="in-views"></a>

### in views

in view templates, use the `@t()` directive. the framework automatically resolves the current locale from the request context.

```html
<h1>@t('common.welcome')</h1>
<button>@t('common.buttons.save')</button>
```

the `@t()` directive is syntactic sugar for calling the `t()` helper with the current request object. you do not need to pass the request manually.

<a name="fallback-behavior"></a>

### fallback behavior

if a translation key is not found, the `t()` function returns the fallback value if one was provided. if no fallback was given, it returns the raw key string itself. this ensures your application never displays blank text due to a missing translation.

```javascript
await t(req, 'common.missing_key');
// returns "common.missing_key"

await t(req, 'common.missing_key', 'fallback text');
// returns "fallback text"
```

if the translation file itself does not exist for the current locale, the function also falls back gracefully.

<a name="variable-interpolation"></a>

### variable interpolation

translations support dynamic parameter placeholders using `:name` syntax matching framework route parameter conventions. pass an object as the third argument to `t()` or second argument to `@t()`.

```json
{
  "greeting": "こんにちは、:nameさん！",
  "items": ":name様は:count個のアイテムを持っています。"
}
```

```javascript
await t(req, 'messages.greeting', { name: '太郎' });
// "こんにちは、太郎さん！"

await t(req, 'messages.items', { name: '山田', count: 5 });
// "山田様は5個のアイテムを持っています。"
```

in views pass the parameter object to `@t()`:

```html
<p>@t('messages.greeting', { name: '太郎' })</p>
```

<a name="japanese-furigana-ruby-support"></a>

### japanese furigana (ruby) support

dframework provides built in support for japanese furigana (ruby text) annotations. you can define furigana directly inside translation strings using `漢字[かんじ]` notation or use the global `ruby()` helper and `@ruby()` directive.

```json
{
  "welcome": "東京[とうきょう]へようこそ、:nameさん！"
}
```

```javascript
await t(req, 'messages.welcome', { name: '花子' });
// "<ruby>東京<rt>とうきょう</rt></ruby>へようこそ、花子さん！"
```

you can also call `ruby()` directly or use `@ruby()` in views:

```javascript
ruby('漢字', 'かんじ');
// "<ruby>漢字<rt>かんじ</rt></ruby>"

ruby('日本[にほん]語[ご]へようこそ');
// "<ruby>日本<rt>にほん</rt></ruby><ruby>語<rt>ご</rt></ruby>へようこそ"
```

```html
<h1>@ruby("漢字", "かんじ")</h1>
<p>@ruby("日本[にほん]語")</p>
```

both `ruby()` and furigana annotated translations output safe html when rendered in views.

<a name="locale-detection"></a>

## locale detection

<a name="detection-order"></a>

### detection order

dframework detects the user's locale automatically using the following priority:

1. **locale cookie** — if a `locale` cookie is present and contains a valid locale code (2-10 alphanumeric characters), it is used immediately
2. **accept-language header** — if no cookie is found, the framework parses the `Accept-Language` http header and uses the primary language code
3. **default locale** — if neither source provides a valid locale, the configured default locale is used

this detection happens automatically during request processing. the detected locale is stored on the request object as `req.locale`.

<a name="default-locale"></a>

### default locale

the default locale is configured in your `config/app.js` file under the `locale` key.

```javascript
// config/app.js
export default {
  locale: 'en'
};
```

if no default is configured, the framework falls back to `'en'`.

<a name="setting-the-locale"></a>

## setting the locale

<a name="persistent-locale"></a>

### persistent locale

use the `Locale.set()` function to change the user's locale. by default, the change is persisted by writing a `locale` cookie and storing the preference in the user's session.

```javascript
import { Locale } from 'dframework';

Locale.set('ja');
```

or use the globally available function:

```javascript
Locale.set('es');
```

the cookie is written immediately to ensure subsequent navigations respect the new locale even before the session is synced. the cookie uses `SameSite=Lax`, the `secure` flag in non local environments, and the root path `/`.

the locale value must match the pattern `/^[a-zA-Z]{2,10}(-[a-zA-Z]{2,8})?$/`. invalid values throw an error.

<a name="request-only-locale"></a>

### request-only locale

if you need to change the locale for only the current request without persisting it, pass `true` as the second argument.

```javascript
Locale.set('fr', true);
```

this sets `req.locale` for the duration of the request but does not write a cookie or update the session. this is useful for previewing content in different languages or for api endpoints that accept a locale parameter.

<a name="reading-the-locale"></a>

### reading the locale

you can read the current locale from the request context. the `Locale` export provides access to locale management methods.

```javascript
import { Locale } from 'dframework';

const currentLocale = Locale.get();
```

you can also read the default locale directly.

```javascript
const defaultLocale = Locale.getDefault();
```

<a name="caching-and-hot-reload"></a>

## caching and hot reload

translation files are cached in memory after their first read. subsequent calls to `t()` with the same locale and file combination return the cached dictionary without hitting the filesystem.

in the `local` environment, the framework sets up file watchers on each loaded translation file. when you edit a translation file and save it, the cache for that specific file and locale is automatically invalidated. the next call to `t()` will reread the updated file from disk. this provides instant feedback during development without restarting the server.

the file watchers are automatically cleaned up when the application shuts down. in production, no file watchers are created and the cache persists for the lifetime of the process.
