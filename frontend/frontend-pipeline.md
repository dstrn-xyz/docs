# frontend pipeline

- [introduction](#introduction)
- [the build pipeline](#the-build-pipeline)
- [view compilation](#view-compilation)
- [css optimization and obfuscation](#css-optimization-and-obfuscation)
- [route compilation](#route-compilation)
- [runtime performance optimizations](#runtime-performance-optimizations)
  - [offscreen resource throttling](#offscreen-resource-throttling)
  - [transparent csrf management](#transparent-csrf-management)
  - [double submission locking](#double-submission-locking)
  - [navigation fast paths and request cancellation](#navigation-fast-paths-and-request-cancellation)
  - [dom morphing and payload hashing](#dom-morphing-and-payload-hashing)
  - [automatic scope lifecycle and cleanup](#automatic-scope-lifecycle-and-cleanup)
  - [accidental navigation guard](#accidental-navigation-guard)

<a name="introduction"></a>

## introduction

unlike traditional frameworks that rely on external tools like webpack or vite, dframework includes its own highly specialized, zero configuration build pipeline and frontend runtime. the build pipeline eliminates unused css, obfuscates class names, and compiles views and routes into fast representations. concurrently, the frontend client runtime (`dstrn.js`) executes transparent optimizations in the browser to streamline network usage, throttle offscreen resources, and prevent memory leaks.

<a name="the-build-pipeline"></a>

## the build pipeline

the pipeline runs automatically when your application boots in the `production` environment. it calculates a fingerprint of your `views` and `routes` directories. if the fingerprint has not changed since the last build (tracked in `storage/framework/pipeline.json`), the pipeline instantly loads the cached artifacts, reducing startup time to milliseconds.

if changes are detected, the pipeline executes its three main phases: view compilation, css optimization, and route compilation.

<a name="view-compilation"></a>

## view compilation

the first phase compiles all `.d` view templates into raw javascript functions (`.dc` files). these compiled files are saved to `storage/framework/views`.

> [!NOTE]
> compiled `.dc` views are signed with a cryptographic hmac in `manifest.json`. at runtime, the framework verifies these signatures to ensure your compiled views have not been tampered with.

<a name="css-optimization-and-obfuscation"></a>

## css optimization and obfuscation

once the views are compiled, the pipeline scans the `.dc` files and your public javascript files to extract every css class name used in your project.

the css optimizer then processes the framework's utility css with the following steps:

1. **tree shaking**: any utility class not found in your views or javascript is completely removed.
2. **obfuscation**: every kept utility class is renamed to a short, random character sequence (e.g. `mt-4` becomes `ab`).
3. **rewriting**: the optimizer directly edits your compiled `.dc` views and public javascript files, replacing the original class names with their obfuscated counterparts.
4. **integrity updates**: the `manifest.json` is automatically updated with the new cryptographic signatures of the modified `.dc` files.
5. **minification**: the base css, obfuscated utilities, icons, components, and your custom css are bundled and minified into a single `public/css/dstrn.css` file.

the optimizer successfully identifies and rewrites classes within standard `class=""` attributes, dynamic bindings like `:class`, `x-bind:class`, and `v-bind:class`, as well as javascript `classList` operations.

a `css-map.json` file is written to the `storage/framework` directory for debugging purposes, mapping the original class names to their obfuscated versions.

### safelisting dynamic classes

when OPTIMIZE_CSS or css.optimize is true, utility classes are tree shaken and obfuscated based on static usage in `.d` views and public javascript. if you construct classes dynamically or generate them at runtime, you can safelist them so they are preserved in the bundle and kept unobfuscated:

specify exact class names, wildcard strings ending in `*`, or regular expressions in the `css.safelist` array in `config/app.js`:

```javascript
export default {
  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
    safelist: ['card', 'badge-*', /^nav-/],
  },
};
```

- exact tokens (such as `card`) preserve only `.card` and its responsive variants (such as `md:card`), without matching compound utilities like `card-header`.
- wildcard tokens (such as `badge-*`) preserve all matching utility classes and their responsive variants (such as `md:badge-pill`).
- template literals with dynamic prefixes (such as ``btn-${size}``) in views and client scripts are automatically detected and preserved without manual configuration.

### opting out of css optimization

if you prefer to skip css minification, tree shaking, and obfuscation entirely, you can opt out by updating your `config/app.js` configuration.

```javascript
export default {
  // ...
  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
  },
};
```

when optimization is disabled, the pipeline will still combine your core and user css files into a single `dstrn.css` payload, but it will not obfuscate or rewrite class names.

<a name="route-compilation"></a>

## route compilation

finally, the `RouteCompiler` scans your route definitions and their corresponding controller handlers. it analyzes the source code to determine exactly which middleware, dependencies, and payload types each route requires.

this data is compiled into a highly optimized route cache (`storage/framework/routes`), completely skipping runtime reflection or dependency resolution on incoming http requests.

<a name="runtime-performance-optimizations"></a>

## runtime performance optimizations

in addition to build time optimizations, the framework runtime in `dstrn.js` applies several performance optimizations transparently in the browser. these optimizations run automatically without requiring manual configuration or developer intervention.

<a name="offscreen-resource-throttling"></a>

### offscreen resource throttling

the runtime initializes an intersection observer with a 120px buffer to monitor major layout sections (`header`, `section`, `article`, `main`, `footer`, `aside`, `.section-frame`, `[d-section]`, `[data-section]`), svg animations (`animate`, `animateMotion`, `animateTransform`), and elements marked with `d-pause-offscreen`.

when an observed element moves out of the viewport:

1. the element receives the `d-paused` attribute.
2. a `d-pause` custom event is dispatched on the element.
3. any active svg animations inside the element are paused via `pauseAnimations()`.
4. any playing html5 `<video>` or `<audio>` elements are paused, and marked for automatic resumption.

when the element reenters the viewport:

1. the `d-paused` attribute is removed.
2. a `d-resume` custom event is dispatched on the element.
3. svg animations resume automatically via `unpauseAnimations()`.
4. media elements that were paused by the optimizer automatically resume playback.

elements containing `<canvas>` or `<d-drawer>` elements are automatically excluded from offscreen throttling. you can also opt out any element or subtree by adding the `d-no-optimize` attribute.

<a name="transparent-csrf-management"></a>

### transparent csrf management

the runtime wraps `window.fetch` to intercept all same origin mutating http requests (`POST`, `PUT`, `PATCH`, `DELETE`). if a request does not already contain an `X-CSRF-TOKEN` header, the runtime extracts the token from the `<meta name="csrf-token">` tag in the document head and attaches it automatically.

on single page application navigations, the runtime extracts updated `csrf-token` and `shield-challenge` meta tags from the server response and synchronizes them with the current document head without requiring manual state management.

<a name="double-submission-locking"></a>

### double submission locking

forms powered by `<d-form>` automatically prevent accidental duplicate submissions and race conditions. upon dispatching the submit event, the form sets an internal submission lock and disables all nested interactive controls (`input`, `textarea`, `select`, `button`).

the controls remain disabled throughout validation and network transport, and are reenabled only after the response is processed or validation errors are displayed.

<a name="navigation-fast-paths-and-request-cancellation"></a>

### navigation fast paths and request cancellation

the navigation runtime optimizes network activity during single page application transitions:

1. **request deduplication**: concurrent navigation requests to the same destination url reuse the active in flight promise rather than triggering duplicate network requests.
2. **request cancellation**: starting a new navigation automatically aborts any preceding in flight navigation fetch using `AbortController`, preventing outdated responses from overwriting newer page content.
3. **same page hash fast path**: clicking an anchor pointing to a hash on the current page (e.g. `/#features` while on `/`) skips the network fetch entirely, scrolls the target element into view, and synchronizes browser history.

<a name="dom-morphing-and-payload-hashing"></a>

### dom morphing and payload hashing

when updating page content, the runtime avoids destructive inner html replacements by using an in place dom morphing algorithm. the morpher updates only changed attributes, text nodes, and form control states (`input`, `textarea`, `select`, `option`), preserving active element focus, text selection, and scroll positions.

for real time reactive updates:

1. **payload hashing**: incoming `d-wire:render` updates calculate a 32 bit hash of the rendered html string (`__wire_last_hash`). if the payload matches the current hash, dom morphing is bypassed completely.
2. **surgical update debouncing**: `d-live` model updates are debounced by 100 milliseconds and suppressed for 250 milliseconds after a successful form submission to prevent redundant refetches.

<a name="automatic-scope-lifecycle-and-cleanup"></a>

### automatic scope lifecycle and cleanup

every swapped view container executes scripts inside an isolated, lifecycle aware scope. a persistent `MutationObserver` monitors the document tree for element removals.

when a container element is removed from the dom, its scope is torn down automatically. the runtime clears all active timeouts (`clearTimeout`), intervals (`clearInterval`), animation frames (`cancelAnimationFrame`), aborts pending scoped `fetch` requests via `AbortController`, and unbinds all attached event listeners and socket subscriptions.

in local development mode, the runtime monitors scope survival across navigations, warning you if an orphan scope persists without being tied to a valid dom node.

<a name="accidental-navigation-guard"></a>

### accidental navigation guard

the runtime registers a global keyboard handler for the backspace key. if the user presses backspace while focus is not on an editable form control (such as a text input, textarea, or `isContentEditable` element), the default browser history back navigation is prevented to protect against accidental data loss.