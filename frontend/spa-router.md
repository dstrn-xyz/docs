# spa router

- [spa router](#spa-router)
  - [introduction](#introduction)
  - [how it works](#how-it-works)
  - [default swap targets](#default-swap-targets)
  - [links and navigation](#links-and-navigation)
    - [anchor interception](#anchor-interception)
    - [the d-link element](#the-d-link-element)
    - [forcing full page navigation on d-link](#forcing-full-page-navigation-on-d-link)
  - [programmatic navigation](#programmatic-navigation)
  - [swap modes](#swap-modes)
  - [transition hooks](#transition-hooks)
    - [global transitions](#global-transitions)
    - [per navigation transitions](#per-navigation-transitions)
    - [hook reference](#hook-reference)
  - [forms](#forms)
    - [basic usage](#basic-usage)
    - [validation](#validation)
    - [json responses](#json-responses)
    - [html responses](#html-responses)
    - [redirect handling](#redirect-handling)
    - [force reload](#force-reload)
    - [callbacks](#callbacks)
    - [programmatic submission](#programmatic-submission)
  - [script execution](#script-execution)
    - [scoped scripts](#scoped-scripts)
    - [persistent scripts](#persistent-scripts)
    - [module scripts](#module-scripts)
    - [external scripts](#external-scripts)
    - [persistent elements (d-spa-keep)](#persistent-elements-d-spa-keep)
  - [active navigation](#active-navigation)
  - [csrf token management](#csrf-token-management)
  - [dom morphing](#dom-morphing)
  - [scope lifecycle](#scope-lifecycle)
  - [scroll behavior](#scroll-behavior)
  - [debug mode](#debug-mode)

<a name="introduction"></a>

## introduction

dframework includes a fully integrated client side router called dSPA. it transforms your server rendered application into a single page application without requiring a javascript framework, a build step, or any configuration. `<a>` tags are intercepted only when marked with the `d-link` attribute. `<d-link>` custom elements are always intercepted. the router fetches the destination, replacing only the targeted portion of the dom while preserving the rest of the page, layout scripts, socket connections, and audio playback.

the router is loaded automatically when your application includes the framework's frontend runtime. there is nothing to install or import.

<a name="how-it-works"></a>

## how it works

when a user clicks a link, the router fetches the destination page as html in the background. it then parses the response into a virtual document, extracts the content inside the configured swap target selectors, and surgically replaces only those regions of the current page. the browser url and history are updated via `pushState`, the document title is synced, csrf tokens are refreshed, and any scripts in the new content are executed in an isolated scope.

back and forward browser navigation is fully handled. when the user presses the back button, the router fetches the previous url and swaps the content exactly as it would for a forward navigation.

<a name="default-swap-targets"></a>

## default swap targets

by default, the router dynamically checks `dSPA.targets` and `window.dSPA_DEFAULT_TARGETS`, falling back to candidate selectors `['#spa-container', 'main', 'body']`. the router resolves the first selector that exists in both the current page and the fetched page. you override this globally by setting `dSPA.targets` or `window.dSPA_DEFAULT_TARGETS`.

```html
<script defer>
  dSPA.targets = ['main'];
</script>
```

<a name="links-and-navigation"></a>

## links and navigation

<a name="anchor-interception"></a>

### anchor interception

standard `<a>` tags are intercepted only when marked with the `d-link` attribute. when clicked, the router fetches the href and swaps the default target region.

```html
<a href="/dashboard" d-link>dashboard</a>
```

the router skips interception in the following cases:

- the link is a standard `<a>` without the `d-link` attribute
- the link points to a different origin
- the link has `target="_blank"`
- the link starts with `#`, `mailto:`, `tel:`, or `javascript:`
- the user is holding a modifier key (ctrl, meta, shift, alt)

<a name="the-d-link-element"></a>

### the d-link element

`<d-link>` is a custom element that behaves identically to an intercepted `<a>` tag but provides additional control over targeting and swap behavior through its attributes.

```html
<d-link href="/dashboard">dashboard</d-link>
```

you can specify which part of the page should be replaced using the `target` attribute.

```html
<d-link href="/admin/users" target="#spa-container">users</d-link>
```

if the fetched page uses different selectors for its content regions, you can specify which element to extract from the response using the `src` attribute.

```html
<d-link href="/settings" target="#panel" src="#settings-panel">settings</d-link>
```

the `mode` attribute controls how the new content is placed into the target element.

```html
<d-link href="/notifications" target="#feed" mode="prepend">load new</d-link>
```

to keep the scroll position after navigation, add the `preserve-scroll` attribute.

```html
<d-link href="/feed?page=2" target="#feed" preserve-scroll>next page</d-link>
```

<a name="forcing-full-page-navigation-on-d-link"></a>

### forcing full page navigation on d-link

standard `<a>` tags without `d-link` already perform normal page navigation. to force a full page reload on a `<d-link>` element, use the `d-full-reload` attribute.

```html
<d-link href="/logout" d-full-reload>logout</d-link>
```

<a name="programmatic-navigation"></a>

## programmatic navigation

you can trigger a spa navigation from javascript using `dSPA.navigate()`. this performs the same fetch, swap, and history update cycle as clicking a link.

```javascript
dSPA.navigate('/dashboard');
```

you can pass an options object to control the target, source, mode, and transitions for the navigation.

```javascript
dSPA.navigate('/home', {
  targets: ['#tabContent'],
  transitions: {
    beforeSwapTarget(oldEl, meta) {
      oldEl.style.opacity = '0';
    },
    afterSwapTarget(newEl, meta) {
      newEl.style.opacity = '1';
    }
  }
});
```

the available options are:

- `targets`: array of css selectors for the elements to replace on the current page
- `srcTargets`: array of css selectors for the elements to extract from the fetched page (defaults to `targets`)
- `mode`: swap mode (`inner`, `outer`, `append`, `prepend`)
- `preserveScroll`: boolean, keeps the scroll position after swap
- `transitions`: object containing transition hook functions
- `addToHistory`: boolean, whether to push a new history entry (defaults to `true`)

<a name="swap-modes"></a>

## swap modes

the router supports four swap modes that control how new content is placed into the target element.

**inner** (default): clears the target element's children and inserts the new content's children into it. the target element itself is preserved.

**outer**: replaces the entire target element with the new content element, including the element itself.

**append**: appends the new content as the last child of the target element without removing existing children.

**prepend**: inserts the new content as the first child of the target element without removing existing children.

<a name="transition-hooks"></a>

## transition hooks

the router exposes lifecycle hooks that let you animate content transitions during navigation. hooks are async aware. the router will wait for any returned promise to resolve before proceeding.

<a name="global-transitions"></a>

### global transitions

you set global transition hooks by assigning an object to `dSPA.transitions` in your layout. these hooks will fire on every spa navigation.

```html
<script defer>
  const drawer = select('d-drawer');

  dSPA.transitions = {
    beforeSwap(oldNode, meta) {
      return drawer.close();
    },
    afterSwap(newNode, meta) {
      return drawer.open();
    },
  };
</script>
```

the `beforeSwap` hook fires before the old content is removed. use it to animate the old content out. the `afterSwap` hook fires after the new content has been inserted. use it to animate the new content in.

<a name="per-navigation-transitions"></a>

### per navigation transitions

you can override the global hooks for a single navigation by passing a `transitions` object to `dSPA.navigate()`.

```javascript
dSPA.navigate('/' + tab.dataset.tab, {
  targets: ['#tabContent'],
  transitions: {
    beforeSwapTarget(oldEl, meta) {
      oldEl.style.transform = 'translate3d(0, 5%, 0)';
      oldEl.style.opacity = '0';
    },
    afterSwapTarget(newEl, meta) {
      newEl.style.transform = 'translate3d(0, 0, 0)';
      newEl.style.opacity = '1';
    }
  }
});
```

<a name="hook-reference"></a>

### hook reference

four hooks are available. each receives two arguments: the dom element being swapped, and a `meta` object containing `url`, `link`, `src`, `dest`, and `mode`.

- `beforeSwap(rootNode, meta)`: fires once before the swap begins, receives the root target element
- `beforeSwapTarget(oldNode, meta)`: fires once per target selector before that specific target is swapped
- `afterSwap(rootNode, meta)`: fires once after the swap is complete and scripts have executed, receives the new root content element
- `afterSwapTarget(newNode, meta)`: fires once per target selector after that specific target has been swapped

if a hook returns a promise, the router directly awaits it before continuing the lifecycle. per navigation transitions passed to `dSPA.navigate()` override global `dSPA.transitions` for that navigation.

<a name="forms"></a>

## forms

<a name="basic-usage"></a>

### basic usage

`<d-form>` is a custom element that submits forms via fetch instead of a full page navigation. it wraps a standard `<form>` element internally and intercepts its submit event.

```html
<d-form action="/register" method="POST" class="flex-column g-1">
  <input type="text" name="username" placeholder="username">
  <input type="password" name="password" placeholder="password">
  <button type="submit">register</button>
</d-form>
```

the element automatically injects the csrf token into every request. if you already have a `<form>` element inside the `<d-form>`, the element will use it directly. if you omit it, a `<form>` is created for you using the `method` and `action` attributes from the `<d-form>`.

all form inputs are disabled during submission to prevent duplicate requests.

<a name="validation"></a>

### validation

you can define client side validation rules using the `rules` attribute. the syntax mirrors the server side validation rules.

```html
<d-form action="/register" method="POST" rules='{"invite_token": "required"}'>
  <input type="text" name="invite_token" placeholder="invite token">
  <button type="submit">submit</button>
</d-form>
```

the available client side rules are: `required`, `string`, `number`, `integer`, `boolean`, `array`, `email`, `min:value`, `max:value`, `in:a,b,c`, `not_in:a,b,c`, `regex:pattern`, and `nullable`.

when validation fails, the form adds an `error` class to the nearest `.input-wrapper` ancestor (or the input itself) and inserts an `error-message` element below it. the errors are cleared automatically on the next submission attempt.

<a name="json-responses"></a>

### json responses

when the server responds with json, the form inspects the response body for specific properties.

if the response contains a `redirect` property, the form navigates to that url via the spa router. if the response also contains a `force: true` property, the form performs a full page redirect instead.

if the response contains an `errors` object, the form displays each error message next to its corresponding input field, following the same convention as client side validation.

<a name="html-responses"></a>

### html responses

when the server responds with html, the form replaces the content of its target element with the response. the target defaults to `body` but can be overridden using the `target` attribute.

```html
<d-form action="/search" method="GET" target="#results">
  <input type="text" name="q" placeholder="search">
  <button type="submit">search</button>
</d-form>
```

if the `navigate` attribute is present, the form uses the spa router's full navigation pipeline instead of a simple content replacement.

```html
<d-form action="/search" method="GET" target="#results" navigate>
  <input type="text" name="q" placeholder="search">
</d-form>
```

<a name="redirect-handling"></a>

### redirect handling

if the server responds with an http redirect (3xx), the form follows it through the spa router, swapping the new page content into the default targets and updating the browser url.

<a name="force-reload"></a>

### force reload

the `force-reload` attribute tells the form to trigger a full page reload under certain conditions instead of handling the response through the spa router.

```html
<!-- always reload after submission -->
<d-form action="/save" method="POST" force-reload>

<!-- reload only on success -->
<d-form action="/register" method="POST" force-reload="success">

<!-- reload only on error -->
<d-form action="/save" method="POST" force-reload="error">
```

the valid values are `true` (or empty string, which is equivalent), `all`, `success`, and `error`.

<a name="callbacks"></a>

### callbacks

you can execute a javascript function after a successful response by specifying its name in the `callback` attribute. the function receives the parsed response data as its argument.

```html
<d-form action="/api/magic" method="POST" callback="onMagicSuccess">
  <button type="submit">generate</button>
</d-form>

<script>
  function onMagicSuccess(data) {
    // handle the response
  }
</script>
```

<a name="programmatic-submission"></a>

### programmatic submission

you can submit a form programmatically by calling `.submit()` on the `<d-form>` element.

```javascript
select('d-form').submit();
```

you can also retrieve the current form data as a `FormData` object by calling `.serialize()`.

```javascript
const data = select('d-form').serialize();
```

<a name="script-execution"></a>

## script execution

the router executes all `<script>` tags found inside swapped content after each navigation. every script runs inside an isolated scope that provides lifecycle aware replacements for timers, listeners, fetch, and animation frames. when the content that owns the script is removed from the dom during a subsequent navigation, all of its timers, listeners, and pending requests are automatically cleaned up.

<a name="scoped-scripts"></a>

### scoped scripts

inline scripts inside swapped content receive scoped versions of standard browser apis as local variables. you use them exactly as you would their global counterparts, but they are automatically torn down when the content is navigated away from.

the following scoped apis are available inside every inline script:

- `setTimeout`, `setInterval`, `clearTimeout`, `clearInterval`
- `requestAnimationFrame`, `cancelAnimationFrame`
- `fetch`
- `listen(target, event, handler, options)`: scoped `addEventListener`
- `listenAll(targets, event, handler, options)`: scoped `addEventListener` on multiple elements
- `unlisten(target, event, handler, options)`: manual `removeEventListener`
- `unlistenAll(targets, event, handler, options)`: manual `removeEventListener` on multiple elements
- `nextFrame()`: returns a promise that resolves on the next animation frame
- `sleep(ms)`: returns a promise that resolves after the specified delay
- `dSPA_SCOPE`: the raw scope object, for advanced usage
- `Socket`: the framework's socket facade
- `dSPA`: the router itself

```html
<div id="counter">0</div>
<script>
  let count = 0;
  const el = document.querySelector('#counter');

  setInterval(() => {
    count++;
    el.textContent = count;
  }, 1000);
</script>
```

when the user navigates away, the interval is automatically cleared. no manual cleanup is required.

<a name="persistent-scripts"></a>

### initial page load and persistent scripts

for standard page views and swapped content, you can write regular `<script>` tags. all inline and external scripts inside swapped regions are automatically managed and scoped by the router.

> [!TIP]
> on the initial hard page load, use `type="text/dspa"` for layout scripts that need access to the managed scope and active websocket connection. regular views navigated via dspa execute scripts inside managed scopes automatically.

```html
<script type="text/dspa">
  Socket.on('update:level', (data) => {
    selectAll('[data-level]').forEach(el => {
      el.textContent = data.level;
    });
  });
</script>
```

the browser parser ignores `type="text/dspa"` on raw page load. on `DOMContentLoaded`, after `Socket.connect()` completes, the router executes all `type="text/dspa"` scripts within the initial managed scope, providing scoped utilities like `Socket`, `listen`, and auto cleaning timers.

for regular views and swapped content navigated via dSPA, standard `<script>` tags are already executed in managed scopes automatically and do not require `type="text/dspa"`.

<a name="module-scripts"></a>

### module scripts

es module scripts (`type="module"`) are supported. by default, module scripts do not receive the scoped api variables because module scope isolation prevents the wrapping technique used for classic scripts.

to opt a module script into scope injection, add the `d-spa-scope` attribute.

```html
<script type="module" d-spa-scope>
  const el = document.querySelector('#timer');
  setInterval(() => {
    el.textContent = new Date().toLocaleTimeString();
  }, 1000);
</script>
```

<a name="external-scripts"></a>

### external scripts

external scripts (those with a `src` attribute) inside swapped content are automatically reexecuted on every spa navigation in document order. the router loads and executes them before running subsequent page scripts.

to prevent the router from executing a specific script tag on spa navigation, add the `d-spa-ignore` attribute.

```html
<script src="/analytics.js" d-spa-ignore></script>
```

<a name="persistent-elements"></a>

### persistent elements (d-spa-keep)

elements and scripts that need to be preserved across body swaps (such as dev tools or persistent layout state) use the `d-spa-keep` attribute. when a body swap occurs, all elements with `d-spa-keep` are retained and reattached to the new body.

```html
<div id="player-container" d-spa-keep>
  <!-- preserved across body swaps -->
</div>
```

<a name="active-navigation"></a>

## active navigation

the router provides a declarative way to highlight the currently active navigation link. add the `d-nav-active` attribute to any element to specify which css classes should be toggled based on the current url.

```html
<d-link
  href="/home"
  d-nav-url="/home"
  d-nav-active="d-link-active"
  class="text-content bdr hover:bg-container-l tr-03 px-1 py-075 flex-center">
  <i class="dstrn-home"></i>
</d-link>
```

`d-nav-url` specifies the url path this element corresponds to. if omitted on elements with `href` (such as `<d-link>` or `<a d-link>`), the router falls back to reading `href`. leading and trailing slashes are normalized automatically, so `/library/`, `library`, and `/library` all match consistently.

by default, the matching is prefix based on path segments. a `d-nav-url` of `/home` will match `/home` and `/home/settings` while ignoring `/home-feed`. to require an exact match, add the `d-nav-exact` attribute.

the active state is recalculated after every spa navigation automatically.

<a name="csrf-token-management"></a>

## csrf token management

the router automatically manages csrf tokens across navigations. when a page is fetched, the router extracts the `csrf-token` and `shield-challenge` meta tags from the response and updates the current page's meta tags with the new values.

all mutating fetch requests (`POST`, `PUT`, `PATCH`, `DELETE`) that target the same origin automatically have the `X-CSRF-TOKEN` header injected. this applies to both the global `fetch()` function and the scoped `fetch()` provided to inline scripts. you never need to manually attach a csrf token to your client side requests.

<a name="dom-morphing"></a>

## dom morphing

during a swap, the router does not blindly replace the old dom with the new dom. it uses a morphing algorithm that walks both trees and applies the minimum number of changes needed to transform the old tree into the new one. this preserves focus state, input values, scroll positions within nested containers, and any runtime state attached to dom nodes that did not change.

the morphing algorithm handles element attribute updates, text node changes, child reordering by id, and input/select/textarea value synchronization.

custom elements that are registered with `customElements.define` are properly upgraded after morphing. if a custom element is encountered, the old element is fully replaced with the new one and upgraded via `customElements.upgrade()`.

<a name="scope-lifecycle"></a>

## scope lifecycle

every time the router executes scripts inside swapped content, it creates a scope. the scope owns all timers, animation frames, fetch requests, and event listeners created by those scripts. when the dom node that owns the scope is removed (either by a subsequent navigation or by direct dom manipulation), a `MutationObserver` detects the removal and automatically tears down the scope, clearing every timer, aborting every pending fetch, and removing every event listener.

this means you never need to write cleanup code in your page scripts. the router handles it for you.

<a name="scroll-behavior"></a>

## scroll behavior

by default, the router scrolls to the top of the page after every navigation. if the target url includes a `#hash` fragment (for example `/profile#settings` or `{{ route('home') }}#native`), the router automatically finds the matching element and scrolls it into view.

when clicking a link pointing to a `#hash` on the current page (such as `/#layers` while already on `/`), the router intercepts the navigation, skips the unnecessary network fetch, updates the browser history, and smoothly scrolls the target element into view.

> [!TIP]
> for section links in persistent layouts, use full route urls with the hash (e.g. `{{ route('home') }}#native`) instead of relative hash fragments (`#native`) so navigation works from any page.

you disable scrolling per navigation by adding the `preserve-scroll` attribute to a `<d-link>` or by passing `preserveScroll: true` to `dSPA.navigate()`.

<a name="debug-mode"></a>

## debug mode

in the `local` environment, the framework automatically enables debug mode on the router. debug mode logs detailed information about each navigation, swap timing, scope creation and cleanup, and potential memory leaks.

the router tracks how many navigations each scope has survived. if a scope survives more than five navigations without being cleaned up, a warning is logged indicating a potential memory leak. orphan scopes (scopes no longer linked to any dom node) are also reported.

you can manually toggle debug mode.

```javascript
dSPA.debug = true;
```
