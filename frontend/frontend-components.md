# components

- [components](#components)

  - [introduction](#introduction)

  - [built in components](#built-in-components)

    - [d-checkbox](#d-checkbox)
    - [d-color-picker](#d-color-picker)
    - [d-combobox](#d-combobox)
    - [d-context-menu](#d-context-menu)
    - [d-drawer](#d-drawer)
    - [d-dropdown](#d-dropdown)
    - [d-hamburger](#d-hamburger)
    - [d-hold-button](#d-hold-button)
    - [d-icon-button](#d-icon-button)
    - [d-image-input](#d-image-input)
    - [d-file-input](#d-file-input)
    - [d-loader](#d-loader)
    - [d-modal](#d-modal)
    - [d-notification](#d-notification)
    - [d-skeleton](#d-skeleton)
    - [d-slider](#d-slider)
    - [d-text-input](#d-text-input)
    - [d-toggle](#d-toggle)
    - [d-morph](#d-morph)

  - [overriding built in components](#overriding-built-in-components)

    - [immutable base class](#immutable-base-class)

  - [dComponent (base class)](#dcomponent-base-class)

    - [rendering strategy](#rendering-strategy)
    - [reactive state](#reactive-state)
    - [auto cleanup api](#auto-cleanup-api)
    - [dom caching](#dom-caching)
    - [inner content capture](#inner-content-capture)
    - [internal event emitter](#internal-event-emitter)
    - [websocket](#websocket)

<a name="introduction"></a>

## introduction

dframework ships a set of web components and a performance tailored base class for authoring custom ones. components are standard custom elements with no virtual dom, no compiler, no build step. the framework compiles and bundles component files in `public/js/components/` automatically.

<a name="built-in-components"></a>

## built in components

### d-checkbox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/checkbox.mp4" type="video/mp4">
  </video>
</p>

```html
<d-checkbox text="enable notifications" checked></d-checkbox>
```

| attribute | description           |
| --------- | --------------------- |
| `text`    | label text            |
| `checked` | initial checked state |

events: `change`

### d-color-picker

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/color-picker.mp4" type="video/mp4">
  </video>
</p>

```html
<d-color-picker value="#ff0000" name="theme color"></d-color-picker>
```

methods: `toggle()`, `setHex(hex)`
events: `change`

### d-combobox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/combobox.mp4" type="video/mp4">
  </video>
</p>

a searchable, extensible select component.

```html
<d-combobox
  options='[{"value":"a","text":"option A"},{"value":"b","text":"option B"}]'
  placeholder="select an option"
  allow-search>
</d-combobox>
```

| attribute      | description                                 |
| -------------- | ------------------------------------------- |
| `options`      | json array of `{value, text}` objects       |
| `placeholder`  | placeholder text                            |
| `allow-search` | enable search filtering                     |
| `allow-input`  | allow custom values not in the options list |

methods: `addOption(value, text)`, `removeOption(value)`, `clearOptions()`
events: `change`, `add`, `open`, `close`

### d-context-menu

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/context-menu.mp4" type="video/mp4">
  </video>
</p>

renders a context menu on right click within the parent element:

```html
<div>
  right-click anywhere here
  <d-context-menu>
    <span>copy</span>
    <span>paste</span>
    <span>delete</span>
  </d-context-menu>
</div>
```

### d-drawer

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/drawer.mp4" type="video/mp4">
  </video>
</p>

```html
<d-drawer direction="left" opened>
  <nav>navigation content</nav>
</d-drawer>
```

| attribute   | description                      |
| ----------- | -------------------------------- |
| `direction` | `top`, `right`, `bottom`, `left` |
| `opened`    | initial open state               |

methods: `toggle()`

### d-dropdown

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/dropdown.mp4" type="video/mp4">
  </video>
</p>

```html
<d-dropdown header="account">
  <a href="/profile">profile</a>
  <a href="/settings">settings</a>
  <a href="/logout" full>log out</a>
</d-dropdown>
```

### d-hamburger

responsive mobile navigation hamburger and fullscreen overlay with no duplicate markup, programmable breakpoints, and subcategory navigation:

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <a d-link href="/" class="flex-row align-center">
      <img src="/img/logo.png" alt="logo">
    </a>

    <div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
      <a d-link href="/features" class="nav-link">features</a>
      <a d-link href="/pricing" class="nav-link">pricing</a>
    </div>

    <d-hamburger target="#nav-links" breakpoint="md">
      <div slot="header">
        <img src="/img/logo.png" class="logo-mark">
      </div>
      <div slot="footer">
        <d-text-input placeholder="search docs"></d-text-input>
      </div>
    </d-hamburger>
  </div>
</nav>
```

#### subcategory navigation (nested groups)

wrap related links in a container with `d-submenu="category title"` inside the desktop navigation. the component automatically transforms it into a chevron trigger on mobile that slides into the subcategory view:

```html
<div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
  <a d-link href="/">home</a>

  <div d-submenu="framework">
    <a d-link href="/layers">architecture</a>
    <a d-link href="/benchmarks">benchmarks</a>
    <a d-link href="/native">native runtime</a>
  </div>

  <div d-submenu="resources">
    <a d-link href="/colors">colors</a>
    <a d-link href="/icons">icons</a>
  </div>
</div>
```

#### subcategory navigation (linked selector)

use `d-submenu="#element-id"` on a link to point to a separate submenu container:

```html
<div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
  <a d-link href="/">home</a>
  <a href="#" d-submenu="#docs-menu">documentation</a>
</div>

<div id="docs-menu" class="d-none">
  <a d-link href="/docs/models">models</a>
  <a d-link href="/docs/views">views</a>
  <a d-link href="/docs/routing">routing</a>
</div>
```

#### standalone custom menu

pass custom markup and slots directly inside `<d-hamburger>` without a `target` attribute:

```html
<d-hamburger breakpoint="md">
  <div slot="header">
    <span class="bold">navigation</span>
  </div>

  <a d-link href="/dashboard">dashboard</a>
  <div d-submenu="settings">
    <a d-link href="/profile">profile</a>
    <a d-link href="/security">security</a>
  </div>

  <div slot="footer">
    <button class="btn btn-accent w-100">sign out</button>
  </div>
</d-hamburger>
```

| attribute        | description                                                               |
| ---------------- | ------------------------------------------------------------------------- |
| `target`         | css selector of desktop container to clone links from (e.g. `#nav-links`) |
| `breakpoint`     | viewport breakpoint (`sm`, `md`, `lg`, `xl`, `xxl`, `ultra` or `48em`)    |
| `opened`         | two way boolean state for open and close                                  |
| `animation`      | overlay transition (`slide-down`, `fade`, `slide-right`, `slide-left`)    |
| `direction`      | entrance direction (`top`, `right`, `left`, `bottom`)                     |
| `size`           | button size in em units (defaults to `1.5em`)                             |
| `color`          | button stroke color (defaults to `currentColor`)                          |
| `activecolor`    | active button color when open (defaults to `var(--accent)`)               |
| `autohidetarget` | automatically hide target element on mobile (defaults to `true`)          |
| `title`          | default title in overlay header when `slot="header"` is omitted           |

slots: `slot="header"`, `slot="footer"`\
methods: `open()`, `close()`, `toggle()`, `navigateTo(panel, title)`, `navigateBack()`\
events: `open`, `close`

### d-hold-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/hold-button.mp4" type="video/mp4">
  </video>
</p>

fires `click` only after the user holds for the specified duration. prevents accidental activation of destructive actions.

```html
<d-hold-button delay="2000">
  delete
</d-hold-button>
```

### d-icon-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/icon-button.mp4" type="video/mp4">
  </video>
</p>

```html
<d-icon-button icon="dstrn-heart" size="2em" color="red"></d-icon-button>
```

### d-image-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/image-input.mp4" type="video/mp4">
  </video>
</p>

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="contain"></d-image-input>
```

| attribute      | description                                                      |
| -------------- | ---------------------------------------------------------------- |
| `accept`       | allowed file types (default `image/png, image/jpeg, image/gif`)  |
| `icon`         | placeholder icon class (default `dstrn-picture`)                 |
| `name`         | input name                                                       |
| `id`           | input id                                                         |
| `placeholder`  | primary dropzone label text (default `drag & drop image`)        |
| `subtitle`     | secondary dropzone helper text (default `or click to browse`)    |
| `replace-text` | label text for the replace button (default `replace`)            |
| `delete-text`  | label text for the delete button (default `delete`)              |
| `no-replace`   | disables replace action (default `false`)                        |
| `no-delete`    | disables delete action (default `false`)                         |
| `fit`          | object fit property applied to preview image (default `contain`) |
| `value`        | file object or url string path                                   |

methods: `reset()`
events: `change`

### d-file-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/file-input.mp4" type="video/mp4">
  </video>
</p>

```html
<d-file-input accept="*/*" name="document" multiple compact></d-file-input>
```

| attribute     | description                                                   |
| ------------- | ------------------------------------------------------------- |
| `accept`      | allowed file types (default `*/*`)                            |
| `icon`        | placeholder icon class (default `dstrn-folder`)               |
| `name`        | input name                                                    |
| `id`          | input id                                                      |
| `placeholder` | primary dropzone label text (default `drag & drop file`)      |
| `subtitle`    | secondary dropzone helper text (default `or click to browse`) |
| `compact`     | render as a slim horizontal bar (default `true`)              |
| `multiple`    | allow selecting multiple files (default `false`)              |
| `value`       | file object, url string path, or array of files/urls          |

methods: `reset()`
events: `change`

### d-loader

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/loader.mp4" type="video/mp4">
  </video>
</p>

```html
<d-loader></d-loader>
```

### d-modal

```html
<d-modal>
  <h2>confirm action</h2>
  <p>this cannot be undone.</p>
  <button>confirm</button>
</d-modal>
```

methods: `open()`, `close()`

### d-notification

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/notification.mp4" type="video/mp4">
  </video>
</p>

```html
<d-notification timer="4000">
  <p>your changes have been saved.</p>
</d-notification>
```

methods: `show()`, `hide()`, `destroy()`

### d-skeleton

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/skeleton.mp4" type="video/mp4">
  </video>
</p>

placeholder shapes rendered during content loading:

```html
<d-skeleton type="text"   lines="3"></d-skeleton>
<d-skeleton type="circle" size="3em"></d-skeleton>
<d-skeleton type="rect"   width="100%" height="200px"></d-skeleton>
<d-skeleton type="card"></d-skeleton>
```

| attribute          | default | description                          |
| ------------------ | ------- | ------------------------------------ |
| `type`             | —       | `text`, `circle`, `rect`, `card`     |
| `lines`            | `1`     | number of lines (for `text` type)    |
| `size`             | —       | shorthand for equal width and height |
| `width` / `height` | —       | explicit dimensions                  |
| `radius`           | —       | border radius override               |

### d-slider

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/slider.mp4" type="video/mp4">
  </video>
</p>

```html
<d-slider min="0" max="100" value="50" step="1"></d-slider>
```

elements with `data-slider-<id>` automatically update with the current slider percentage:

```html
<d-slider id="volume" name="volume" min="0" max="1" step="0.01" value="0.4"></d-slider>
<div data-slider-volume class="text-content-l" style="width:4em;">40%</div>
```

methods: `getValue()`, `setValue(value)`
events: `change`

### d-text-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/input.mp4" type="video/mp4">
  </video>
</p>

```html
<d-text-input placeholder="search" icon="dstrn-search" shortcut="cmd+k"></d-text-input>
```

| attribute      | description                                   |
| -------------- | --------------------------------------------- |
| `placeholder`  | placeholder text                              |
| `icon`         | input icon class                              |
| `type`         | input type (text, password, search, number)   |
| `shortcut`     | keyboard shortcut to quickly focus the input  |
| `autocomplete` | enable browser autocomplete (default `false`) |
| `disabled`     | disables the input element (default `false`)  |
| `readonly`     | sets the input to read only (default `false`) |

### d-toggle

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/toggle.mp4" type="video/mp4">
  </video>
</p>

renders a segmented toggle between two or more options. mark the default with `default`:

```html
<d-toggle>
  <div default>option 1</div>
  <div>option 2</div>
</d-toggle>
```

events: `change`

### d-morph

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/morph.mp4" type="video/mp4">
  </video>
</p>

standalone morphing elements. the `dMotion` singleton engine orchestrates physics based animated transitions between named elements placed anywhere in the document.

```html
<d-morph name="compact" d-action="long-press" d-to="expanded" d-duration="400" d-visible>
  <div class="compact-card">compact view</div>
</d-morph>

<d-morph name="expanded" d-action="click" d-to="compact" d-duration="300">
  <div class="expanded-card">expanded view</div>
</d-morph>
```

| attribute    | description                                        |
| ------------ | -------------------------------------------------- |
| `name`       | unique identifier for this morph element           |
| `d-visible`  | marks this element as initially visible            |
| `d-action`   | trigger type: `click`, `long-press` or `click-out` |
| `d-to`       | name of the target morph element to transition to  |
| `d-duration` | transition duration in milliseconds                |

methods: `show()`, `hide()`

place `d-no-morph` on child elements to exempt them from the transition and allow standard interaction.

**`window.dMotion`** **api:**

```javascript
dMotion.get('compact')                                  // get a morph instance by name
dMotion.getPrevious()                                   // returns the immediately previous morph, null if no transition has occurred
dMotion.transition('compact', 'expanded', triggerNode)  // trigger a transition programmatically
dMotion.listen('compact', 'expanded', fn, 'before')     // hook: 'before' or 'after'
dMotion.unlisten('compact', 'expanded', fn, 'before')
```

<a name="overriding-built-in-components"></a>

## overriding built in components

every built in component behavior can be customized, extended, or completely overwritten by application code. to override a built in component, place a file with the exact same component filename under `public/js/components/` (such as `public/js/components/dCheckbox.js`).

when compiling the frontend bundle, the compiler automatically detects the user file and replaces the built in component implementation in place.

```javascript
// public/js/components/dCheckbox.js
class dCheckbox extends dComponent {
  static form = true;
  static get tag() { return 'd-checkbox'; }

  // custom template or methods
}

dComponent.define(dCheckbox);
```

### immutable base class

all custom elements and overrides extend `dComponent`. unlike UI components, `dComponent` is the immutable framework base class providing reactivity, reflection, and lifecycle management. the compiler always preserves the core `dComponent` implementation at the root of the bundle so all components inherit from it consistently.

<a name="dcomponent--base-class"></a>

## dComponent (base class)

`dComponent` is the authoring base class for all custom components. it provides property reflection, reactive state, form integration, lifecycle management, and automatic memory cleanup. it does not use a virtual dom, rendering is surgical by design.

```javascript
class dCounter extends dComponent {
  static tag   = 'd-counter'
  static form  = true // opt into native form element integration

  static props = {
    value: { type: 'number',  default: 0, form: true },
    step: { type: 'number',  default: 1 },
    disabled: { type: 'boolean', default: false },
    label: { type: 'string',  default: 'count' },
  }

  template() {
    // called once on mount and returns initial innerHTML
    return `
      <span class="label">${this.label}</span>
      <button class="dec">−</button>
      <span class="value">${this.value}</span>
      <button class="inc">+</button>
    `;
  }

  mount() {
    // called once after template initialisation
    // good for effects that depend on reactive state
    this.effect(() => {
      // reading this.state.value registers it as a dependency
      // the effect reruns whenever state.value changes
      document.title = `count: ${this.state.value}`;
      return () => { document.title = 'myapp'; }; // optional teardown
    });
  }

  render() {
    // called automatically on every prop or state change
    // use for surgical dom updates only (never modify this.state here)
    this.refs('.value')[0].textContent = this.state.value ?? this.value;
    this.refs('button.dec')[0].disabled = this.disabled;
    this.refs('button.inc')[0].disabled = this.disabled;

    // listeners are cleared and rebound between renders (no duplicate handlers)
    this.listen(this.refs('button.dec')[0], 'click', () => this.decrement());
    this.listen(this.refs('button.inc')[0], 'click', () => this.increment());
  }

  onPropChanged(name, oldVal, newVal) {
    if (name === 'value') Log.debug('value changed', oldVal, '->', newVal);
  }

  increment() {
    this.setState(s => { s.value = (s.value ?? this.value) + this.step; });
  }

  decrement() {
    this.setState(s => { s.value = (s.value ?? this.value) - this.step; });
  }
}

dComponent.define(dCounter);
```

### rendering strategy

`dComponent` does not provide a virtual dom. structure belongs in `template()`. all dom mutations belong in `render()` or `onPropChanged()`. this eliminates reflows, preserves listener state, and prevents flicker. any error thrown inside `render()` is caught and logged without crashing the component.

`this.state` cannot be modified inside `render()`, the engine ignores such writes to prevent infinite loops.

### reactive state

```javascript
// update state and trigger render()
this.setState({ active: true });
this.setState(s => { s.count++; s.lastUpdated = Date.now(); });

// update state without triggering render() but still triggers effect() loops
this.setState({ timer: Date.now() }, false);
```

### auto cleanup api

use these instead of native browser apis to prevent memory leaks when the component is removed from the dom:

```javascript
this.listen(target, event, callback, opts?) // addEventListener with cleanup
this.listenAll(targets, event, callback)    // attach to multiple elements
this.setTimeout(callback, ms)
this.setInterval(callback, ms)
this.requestAnimationFrame(callback)
```

### dom caching

```javascript
this.refs(selector) // cached querySelectorAll, returns an array
```

### inner content capture

```javascript
this.originalChildren // array of cloned light dom nodes from before template()
this.originalHTML     // original innerHTML as a string
```

### internal event emitter

```javascript
this.on(event, callback)
this.emit(event, ...args)
```

### websocket

```javascript
this.wire(event, payload) // emit a d-wire event through the global socket loop
```
