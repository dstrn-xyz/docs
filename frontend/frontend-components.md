# frontend components

- [frontend components](#frontend-components)
  - [introduction](#introduction)
  - [built in components](#built-in-components)
    - [d-checkbox](#d-checkbox)
      - [attributes and properties](#attributes-and-properties)
      - [programmatic api](#programmatic-api)
      - [events](#events)
      - [behavior](#behavior)
    - [d-color-picker](#d-color-picker)
      - [attributes and properties](#attributes-and-properties-1)
      - [programmatic api](#programmatic-api-1)
      - [events](#events-1)
      - [behavior](#behavior-1)
    - [d-combobox](#d-combobox)
      - [attributes and properties](#attributes-and-properties-2)
      - [programmatic api](#programmatic-api-2)
      - [events](#events-2)
      - [behavior](#behavior-2)
    - [d-context-menu](#d-context-menu)
      - [attributes and properties](#attributes-and-properties-3)
      - [programmatic api](#programmatic-api-3)
      - [events](#events-3)
      - [behavior](#behavior-3)
    - [d-drawer](#d-drawer)
      - [attributes and properties](#attributes-and-properties-4)
      - [programmatic api](#programmatic-api-4)
      - [events](#events-4)
      - [behavior](#behavior-4)
    - [d-dropdown](#d-dropdown)
      - [attributes and properties](#attributes-and-properties-5)
      - [programmatic api](#programmatic-api-5)
      - [behavior](#behavior-5)
    - [d-file-input](#d-file-input)
      - [attributes and properties](#attributes-and-properties-6)
      - [programmatic api](#programmatic-api-6)
      - [events](#events-5)
      - [behavior](#behavior-6)
    - [d-hamburger](#d-hamburger)
      - [attributes and properties](#attributes-and-properties-7)
      - [slots](#slots)
      - [programmatic api](#programmatic-api-7)
      - [events](#events-6)
      - [behavior](#behavior-7)
    - [d-hold-button](#d-hold-button)
      - [attributes and properties](#attributes-and-properties-8)
      - [programmatic api](#programmatic-api-8)
      - [events](#events-7)
      - [behavior](#behavior-8)
    - [d-icon-button](#d-icon-button)
      - [attributes and properties](#attributes-and-properties-9)
    - [d-image-input](#d-image-input)
      - [attributes and properties](#attributes-and-properties-10)
      - [programmatic api](#programmatic-api-9)
      - [events](#events-8)
      - [behavior](#behavior-9)
    - [d-loader](#d-loader)
      - [attributes and properties](#attributes-and-properties-11)
      - [programmatic api](#programmatic-api-10)
      - [behavior](#behavior-10)
    - [d-modal](#d-modal)
      - [attributes and properties](#attributes-and-properties-12)
      - [programmatic api](#programmatic-api-11)
      - [events](#events-9)
      - [behavior](#behavior-11)
    - [d-morph](#d-morph)
      - [attributes and properties](#attributes-and-properties-13)
      - [programmatic api (`window.dMotion`)](#programmatic-api-windowdmotion)
      - [behavior](#behavior-12)
    - [d-notification](#d-notification)
      - [attributes and properties](#attributes-and-properties-14)
      - [programmatic api](#programmatic-api-12)
      - [behavior](#behavior-13)
    - [d-skeleton](#d-skeleton)
      - [attributes and properties](#attributes-and-properties-15)
    - [d-slider](#d-slider)
      - [attributes and properties](#attributes-and-properties-16)
      - [programmatic api](#programmatic-api-13)
      - [events](#events-10)
      - [behavior](#behavior-14)
    - [d-text-input](#d-text-input)
      - [attributes and properties](#attributes-and-properties-17)
      - [programmatic api](#programmatic-api-14)
      - [events](#events-11)
      - [behavior](#behavior-15)
    - [d-toggle](#d-toggle)
      - [attributes and properties](#attributes-and-properties-18)
      - [programmatic api](#programmatic-api-15)
      - [events](#events-12)
      - [behavior](#behavior-16)
  - [overriding built in components](#overriding-built-in-components)
    - [immutable base class](#immutable-base-class)
  - [dComponent (base class)](#dcomponent-base-class)
    - [component definition and registration](#component-definition-and-registration)
    - [lifecycle methods](#lifecycle-methods)
    - [rendering strategy and surgical updates](#rendering-strategy-and-surgical-updates)
    - [reactive state](#reactive-state)
    - [effects and dependency tracking](#effects-and-dependency-tracking)
    - [automatic cleanup apis](#automatic-cleanup-apis)
    - [dom caching and refs](#dom-caching-and-refs)
    - [inner content capture](#inner-content-capture)
    - [form integration](#form-integration)
    - [internal event emitter](#internal-event-emitter)
    - [websocket integration](#websocket-integration)

<a name="introduction"></a>

## introduction

dframework ships a complete library of zero dependency, high performance custom web components compiled into the frontend bundle. components are standard custom elements with no virtual dom, no compiler requirements, and no client build step. the framework compiles and bundles component files in `public/js/components/` automatically on server boot.

<a name="built-in-components"></a>

## built in components

### d-checkbox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/checkbox.mp4" type="video/mp4">
  </video>
</p>

form associated checkbox switch with synchronized label and native form submission integration.

```html
<d-checkbox text="enable notifications" checked name="notifications" value="1"></d-checkbox>
```

#### attributes and properties

| attribute | property  | type      | default      | form | description                                  |
| :-------- | :-------- | :-------- | :----------- | :--- | :------------------------------------------- |
| `checked` | `checked` | `boolean` | `false`      | yes  | checked state reflected in form submission   |
| `text`    | `text`    | `string`  | `"checkbox"` | no   | label text displayed beside the toggle       |
| `name`    | `name`    | `string`  | `""`         | no   | input field name used during form submission |
| `value`   | `value`   | `string`  | `"on"`       | no   | form submission payload sent when checked    |

#### programmatic api

```javascript
const checkbox = select('d-checkbox');

// get or set checked state
checkbox.checked = true;

// update label text
checkbox.text = 'allow telemetry';

// update form payload name and value
checkbox.name = 'telemetry';
checkbox.value = 'accepted';
```

#### events

| event    | detail              | bubbles | description                                       |
| :------- | :------------------ | :------ | :------------------------------------------------ |
| `change` | native event object | no      | dispatched whenever the user toggles the checkbox |

#### behavior

- automatically synchronizes checked state, label text, and form submission values when toggled by the user or modified via javascript.

---

### d-color-picker

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/color-picker.mp4" type="video/mp4">
  </video>
</p>

high precision color picker featuring 2d saturation and brightness controls, hue slider, and alpha transparency controls.

```html
<d-color-picker value="#3b82f6" name="theme_color" label="accent color"></d-color-picker>
```

#### attributes and properties

| attribute | property | type      | default          | form | description                                             |
| :-------- | :------- | :-------- | :--------------- | :--- | :------------------------------------------------------ |
| `value`   | `value`  | `string`  | `"#d3ac5f"`      | yes  | current hex color value (supports 3, 4, 6, 8 digit hex) |
| `name`    | `name`   | `string`  | `""`             | no   | input name attribute for form submission                |
| `id`      | `id`     | `string`  | `""`             | no   | element identifier                                      |
| `label`   | `label`  | `string`  | `"color picker"` | no   | header label displayed above the picker                 |
| `opened`  | `opened` | `boolean` | `false`          | no   | whether the popup picker panel is initially open        |

#### programmatic api

```javascript
const picker = select('d-color-picker');

// toggle popup panel visibility
picker.toggle();

// programmatically set hex color
picker.setHex('#ef4444');

// read color state
console.log(picker.hex); // current hex string
console.log(picker.h, picker.s, picker.b, picker.a); // hsb and alpha values
```

#### events

| event    | detail     | bubbles | description                                              |
| :------- | :--------- | :------ | :------------------------------------------------------- |
| `change` | hex string | yes     | dispatched when color changes via drag, input, or setHex |
| `input`  | hex string | yes     | dispatched continuously during dragging                  |

#### behavior

- interactive 2D color plane, hue bar, and alpha slider with checkered transparency.
- supports smooth dragging even outside the component bounds.
- automatically redraws on element resize and renders sharply on high resolution displays.
- clicking the reset icon restores the initial color value.

---

### d-combobox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/combobox.mp4" type="video/mp4">
  </video>
</p>

searchable, extensible select combobox with automatic floating placement, keyboard navigation, and dynamic option injection.

```html
<d-combobox
  placeholder="select department"
  allow-search
  allow-input
  name="department">
  <option value="eng" selected>engineering</option>
  <option value="des">product design</option>
  <option value="ops">operations</option>
</d-combobox>
```

#### attributes and properties

| attribute      | property       | type      | default | form | description                                                   |
| :------------- | :------------- | :-------- | :------ | :--- | :------------------------------------------------------------ |
| `placeholder`  | `placeholder`  | `string`  | `""`    | no   | placeholder text displayed when no option is selected         |
| `value`        | `value`        | `string`  | `null`  | yes  | currently selected option value                               |
| `allow-search` | `allowSearch`  | `boolean` | `false` | no   | enables real time search filtering input                      |
| `allow-input`  | `allowInput`   | `boolean` | `false` | no   | allows adding custom options via input and plus button        |
| `horizontal`   | `isHorizontal` | `boolean` | `false` | no   | switches option layout from vertical list to horizontal chips |
| `name`         | `name`         | `string`  | `""`    | no   | form field name                                               |
| `id`           | `id`           | `string`  | `""`    | no   | component id                                                  |
| `options`      | `options`      | `array`   | `[]`    | no   | array of `{value, text, content}` objects                     |

#### programmatic api

```javascript
const combo = select('d-combobox');

// add option dynamically
combo.addOption('mktg', 'marketing');

// remove option by value
combo.removeOption('ops');

// clear all options
combo.clearOptions();

// set selected value programmatically
combo.value = 'eng';

// close all open comboboxes across document
dCombobox.closeAll();
```

#### events

| event    | detail                   | bubbles | description                                    |
| :------- | :----------------------- | :------ | :--------------------------------------------- |
| `change` | selected value string    | yes     | dispatched when an option is selected or added |
| `input`  | selected value string    | yes     | dispatched when selection changes              |
| `add`    | `{value, text, content}` | no      | dispatched when a custom option is added       |
| `open`   | none                     | yes     | dispatched when the dropdown panel opens       |
| `close`  | none                     | yes     | dispatched when the dropdown panel closes      |

#### behavior

- automatically parses child `<option>` tags declared in HTML.
- automatically calculates viewport space to open above or below without overflowing the screen.
- keeps floating coordinates aligned during scrolling and resizing.
- provides full keyboard navigation (arrow keys, home, end, enter, escape).
- displays an empty state indicator when search queries return no matches.

---

### d-context-menu

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/context-menu.mp4" type="video/mp4">
  </video>
</p>

contextual right click menu with automatic boundary clamping and outside click suppression.

```html
<div class="card p-2">
  right click inside this card
  <d-context-menu>
    <span>copy link</span>
    <span>duplicate item</span>
    <span>delete</span>
  </d-context-menu>
</div>
```

#### attributes and properties

| attribute | property | type      | default | form | description                                                     |
| :-------- | :------- | :-------- | :------ | :--- | :-------------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the context menu is open                                |
| `parent`  | `parent` | `string`  | `""`    | no   | selector for target parent element (defaults to parent element) |

#### programmatic api

```javascript
const menu = select('d-context-menu');

// open context menu programmatically (optionally passing pointer event for cursor positioning)
menu.open();
menu.open(pointerEvent);

// close context menu
menu.close();

// toggle context menu
menu.toggle();
```

#### events

| event   | detail          | bubbles | description                                                |
| :------ | :-------------- | :------ | :--------------------------------------------------------- |
| `click` | clicked element | no      | emitted through component emitter when an item is selected |

#### behavior

- opens on right click within its parent container and suppresses the browser default context menu.
- calculates menu dimensions and automatically bounds coordinates so the menu never overflows parent or screen edges.
- clicking outside closes the menu immediately without triggering click actions or links on underlying elements.
- automatically closes any other open context menus before opening.

---

### d-drawer

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/drawer.mp4" type="video/mp4">
  </video>
</p>

offcanvas navigation drawer with animated entrance transitions and wave styling.

```html
<d-drawer direction="left" id="main-drawer">
  <nav class="flex-column g-1 p-2">
    <a href="/dashboard">dashboard</a>
    <a href="/settings">settings</a>
  </nav>
</d-drawer>
```

#### attributes and properties

| attribute   | property    | type      | default  | form | description                                              |
| :---------- | :---------- | :-------- | :------- | :--- | :------------------------------------------------------- |
| `opened`    | `opened`    | `boolean` | `false`  | no   | whether the drawer is open                               |
| `direction` | `direction` | `string`  | `"left"` | no   | entrance edge (`"left"`, `"right"`, `"top"`, `"bottom"`) |

#### programmatic api

```javascript
const drawer = select('d-drawer');

// open drawer (returns promise that resolves after animation completes)
await drawer.open();

// close drawer (returns promise)
await drawer.close();

// toggle drawer
await drawer.toggle();

// inspect animation and open state
console.log(drawer.isOpened, drawer.isAnimating);
```

#### events

| event   | detail | bubbles | description                                     |
| :------ | :----- | :------ | :---------------------------------------------- |
| `open`  | none   | yes     | dispatched when the opening animation completes |
| `close` | none   | yes     | dispatched when the closing animation completes |

#### behavior

- slides into view from the configured edge with multi layer wave animations.
- programmatic `open()`, `close()`, and `toggle()` methods return promises that resolve upon animation completion.

---

### d-dropdown

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/dropdown.mp4" type="video/mp4">
  </video>
</p>

collapsible accordion dropdown container, perfect for faqs.

```html
<d-dropdown header="account security">
  <div class="p-1 flex-column g-1">
    <d-checkbox text="two factor authentication"></d-checkbox>
    <a href="/sessions" class="btn btn-sm">manage active sessions</a>
  </div>
</d-dropdown>
```

#### attributes and properties

| attribute | property | type      | default      | form | description                                      |
| :-------- | :------- | :-------- | :----------- | :--- | :----------------------------------------------- |
| `header`  | `header` | `string`  | `"dropdown"` | no   | title text displayed on the clickable header bar |
| `opened`  | `opened` | `boolean` | `false`      | no   | whether the dropdown is expanded                 |

#### programmatic api

```javascript
const dropdown = select('d-dropdown');

// open dropdown
dropdown.open();

// close dropdown
dropdown.close();

// toggle dropdown
dropdown.toggle();

// inspect open state
console.log(dropdown.isOpened);
```

#### behavior

- clicking the header expands or collapses child content with an animated chevron indicator.
- dynamically updates header text and open state when properties change.

---

### d-file-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/file-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop file upload container with interactive item list, formatted byte size calculations, and compact bar modes.

```html
<d-file-input accept=".pdf,.zip,.csv" name="documents" multiple compact></d-file-input>
```

#### attributes and properties

| attribute     | property      | type             | default                | form | description                                  |
| :------------ | :------------ | :--------------- | :--------------------- | :--- | :------------------------------------------- |
| `accept`      | `accept`      | `string`         | `"*/*"`                | no   | allowed mime types or file extensions        |
| `icon`        | `icon`        | `string`         | `"dstrn-folder-line"`  | no   | icon class displayed in dropzone             |
| `placeholder` | `placeholder` | `string`         | `"drag & drop file"`   | no   | primary dropzone title                       |
| `subtitle`    | `subtitle`    | `string`         | `"or click to browse"` | no   | secondary helper subtitle                    |
| `compact`     | `compact`     | `boolean`        | `true`                 | no   | renders as a slim horizontal bar when true   |
| `multiple`    | `multiple`    | `boolean`        | `false`                | no   | allows selecting and dropping multiple files |
| `name`        | `name`        | `string`         | `""`                   | no   | form payload field name                      |
| `id`          | `id`          | `string`         | `""`                   | no   | component id                                 |
| `value`       | `value`       | `object`/`array` | `null`                 | yes  | `File`, `File[]`, or url string payload      |

#### programmatic api

```javascript
const fileInput = select('d-file-input');

// reset dropzone, clearing value, file list, and native input
fileInput.reset();

// inspect selected files
console.log(fileInput.value); // File instance or File[] array
```

#### events

| event    | detail                      | bubbles | description                                             |
| :------- | :-------------------------- | :------ | :------------------------------------------------------ |
| `change` | `File`, `File[]`, or `null` | yes     | dispatched when files are selected, dropped, or deleted |

#### behavior

- drag and drop file upload with file list display, formatted sizes (Bytes, KB, MB, GB), and delete buttons.
- automatically clears error states when a valid file is dropped.
- integrates with standard form submission.

---

### d-hamburger

responsive mobile navigation hamburger trigger and full screen navigation overlay with programmable breakpoints and multi level sliding subcategory panels.

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <a d-link href="/" class="flex-row align-center">
      <img src="/img/logo.png" alt="logo">
    </a>

    <div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
      <a d-link href="/features" class="nav-link">features</a>
      <div d-submenu="framework">
        <a d-link href="/architecture">architecture</a>
        <a d-link href="/benchmarks">benchmarks</a>
      </div>
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

#### attributes and properties

| attribute        | property         | type      | default           | form | description                                                              |
| :--------------- | :--------------- | :-------- | :---------------- | :--- | :----------------------------------------------------------------------- |
| `opened`         | `opened`         | `boolean` | `false`           | no   | whether the overlay is open                                              |
| `breakpoint`     | `breakpoint`     | `string`  | `"md"`            | no   | breakpoint token (`sm`, `md`, `lg`, `xl`, `xxl`, `ultra`) or media query |
| `target`         | `target`         | `string`  | `""`              | no   | selector of desktop container to clone links from                        |
| `animation`      | `animation`      | `string`  | `"slide-down"`    | no   | overlay transition (`slide-down`, `fade`, `slide-right`, `slide-left`)   |
| `direction`      | `direction`      | `string`  | `"top"`           | no   | entrance direction                                                       |
| `size`           | `size`           | `string`  | `"1.5em"`         | no   | hamburger button size in em units                                        |
| `color`          | `color`          | `string`  | `"currentColor"`  | no   | button stroke color                                                      |
| `activecolor`    | `activeColor`    | `string`  | `"var(--accent)"` | no   | button color when open                                                   |
| `autohidetarget` | `autoHideTarget` | `boolean` | `true`            | no   | automatically hides desktop target container below breakpoint            |
| `title`          | `title`          | `string`  | `""`              | no   | fallback header title when `slot="header"` is omitted                    |

#### slots

- `slot="header"`: custom header content in the mobile overlay.
- `slot="footer"`: custom footer content at the bottom of the overlay.

#### programmatic api

```javascript
const hamburger = select('d-hamburger');

// open overlay (returns promise)
await hamburger.open();

// close overlay (returns promise)
await hamburger.close();

// toggle overlay
await hamburger.toggle();

// navigate into a specific subcategory panel programmatically
hamburger.navigateTo(subPanelElement, 'framework');

// navigate back to previous parent panel
hamburger.navigateBack();

// close all open hamburger instances across the page
dHamburger.closeAll();
```

#### events

| event   | detail | bubbles | description                    |
| :------ | :----- | :------ | :----------------------------- |
| `open`  | none   | yes     | dispatched when overlay opens  |
| `close` | none   | yes     | dispatched when overlay closes |

#### behavior

- automatically displays the mobile button and hides the desktop navigation when below the specified breakpoint.
- supports nested sliding subpanels using `d-submenu` attributes on link containers.
- automatically manages page scroll locking while open.
- closes automatically when clicking links, pressing Escape, or navigating between pages.

---

### d-hold-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/hold-button.mp4" type="video/mp4">
  </video>
</p>

press and hold button that prevents accidental triggers of destructive operations by requiring a continuous pointer press.

```html
<d-hold-button delay="1500" class="btn btn-red" type="submit">
  hold to delete database
</d-hold-button>
```

#### attributes and properties

| attribute  | property   | type      | default    | form | description                                                     |
| :--------- | :--------- | :-------- | :--------- | :--- | :-------------------------------------------------------------- |
| `delay`    | `delay`    | `number`  | `800`      | no   | hold duration in milliseconds required before triggering action |
| `type`     | `type`     | `string`  | `"button"` | no   | button type (`"button"`, `"submit"`)                            |
| `disabled` | `disabled` | `boolean` | `false`    | no   | whether the button is disabled                                  |

#### programmatic api

```javascript
const holdBtn = select('d-hold-button');

// adjust required hold duration in ms
holdBtn.delay = 2000;

// toggle disabled state
holdBtn.disabled = true;
```

#### events

| event   | detail             | bubbles | description                                               |
| :------ | :----------------- | :------ | :-------------------------------------------------------- |
| `click` | native click event | yes     | dispatched only after holding for the full delay duration |

#### behavior

- requires continuous press for the configured duration before triggering the click action.
- releasing before completion cancels the action and resets visual progress.
- automatically submits the surrounding form when configured with `type="submit"`.

---

### d-icon-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/icon-button.mp4" type="video/mp4">
  </video>
</p>

icon button wrapper supporting custom icons, sizes, and dynamic theme active colors.

```html
<d-icon-button icon="dstrn-heart-line" size="2em" color="var(--content-l)" activecolor="var(--red)"></d-icon-button>
```

#### attributes and properties

| attribute     | property      | type     | default              | form | description                                              |
| :------------ | :------------ | :------- | :------------------- | :--- | :------------------------------------------------------- |
| `icon`        | `icon`        | `string` | `""`                 | no   | css icon class (e.g. `dstrn-heart-line`, `dstrn-search`) |
| `size`        | `size`        | `string` | `"4.2em"`            | no   | font size of the icon container                          |
| `color`       | `color`       | `string` | `"var(--content-l)"` | no   | default icon color                                       |
| `activecolor` | `activeColor` | `string` | `"var(--accent)"`    | no   | active icon color                                        |
| `id`          | `id`          | `string` | `""`                 | no   | component id                                             |

---

### d-image-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/image-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop image dropzone with instant client preview and action overlay.

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="cover" placeholder="drop profile photo"></d-image-input>
```

#### attributes and properties

| attribute      | property      | type              | default                              | form | description                                          |
| :------------- | :------------ | :---------------- | :----------------------------------- | :--- | :--------------------------------------------------- |
| `accept`       | `accept`      | `string`          | `"image/png, image/jpeg, image/gif"` | no   | allowed image mime types                             |
| `icon`         | `icon`        | `string`          | `"dstrn-pic-line"`                   | no   | placeholder icon class                               |
| `placeholder`  | `placeholder` | `string`          | `"drag & drop image"`                | no   | dropzone title text                                  |
| `subtitle`     | `subtitle`    | `string`          | `"or click to browse"`               | no   | dropzone subtitle text                               |
| `replace-text` | `replaceText` | `string`          | `"replace"`                          | no   | text for overlay replace button                      |
| `delete-text`  | `deleteText`  | `string`          | `"delete"`                           | no   | text for overlay delete button                       |
| `no-replace`   | `noReplace`   | `boolean`         | `false`                              | no   | hides the replace button when true                   |
| `no-delete`    | `noDelete`    | `boolean`         | `false`                              | no   | hides the delete button when true                    |
| `fit`          | `fit`         | `string`          | `"contain"`                          | no   | object-fit CSS property (`contain`, `cover`, `fill`) |
| `name`         | `name`        | `string`          | `""`                                 | no   | form payload field name                              |
| `id`           | `id`          | `string`          | `""`                                 | no   | component id                                         |
| `value`        | `value`       | `object`/`string` | `null`                               | yes  | `File` object or image URL string                    |

#### programmatic api

```javascript
const imgInput = select('d-image-input');

// reset dropzone and clear image preview
imgInput.reset();

// get or set preview src directly
imgInput.preview = '/img/avatar.png';

// read selected File instance
console.log(imgInput.value);
```

#### events

| event    | detail           | bubbles | description                                               |
| :------- | :--------------- | :------ | :-------------------------------------------------------- |
| `change` | `File` or `null` | yes     | dispatched when an image is selected, dropped, or deleted |

#### behavior

- provides drag and drop image selection with instant preview and action buttons to replace or remove images.
- automatically integrates with form submission and clears error states on upload.

---

### d-loader

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/loader.mp4" type="video/mp4">
  </video>
</p>

high performance canvas arc spinner with automatic viewport intersection pause.

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

#### attributes and properties

| attribute | property | type     | default | form | description                                                  |
| :-------- | :------- | :------- | :------ | :--- | :----------------------------------------------------------- |
| `color`   | `color`  | `string` | `""`    | no   | stroke color (defaults to `--accent` or computed text color) |

#### programmatic api

```javascript
const loader = select('d-loader');

// destroy loader and disconnect observers
loader.destroy();
```

#### behavior

- lightweight canvas spinner that automatically pauses animation when scrolled offscreen to conserve CPU.
- automatically adapts to container dimensions and renders sharply on high resolution screens.

---

### d-modal

modal dialog container with page scroll locking and backdrop dismiss protection.

```html
<d-modal id="confirm-modal">
  <div class="flex-column g-1 p-2">
    <h3>confirm deletion</h3>
    <p class="text-content-l">this action cannot be undone.</p>
    <div class="flex-row justify-end g-1 mt-1">
      <button class="btn btn-sm" onclick="this.closest('d-modal').close()">cancel</button>
      <button class="btn btn-sm btn-red">delete</button>
    </div>
  </div>
</d-modal>
```

#### attributes and properties

| attribute | property | type      | default | form | description                            |
| :-------- | :------- | :-------- | :------ | :--- | :------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the modal is currently visible |

#### programmatic api

```javascript
const modal = select('d-modal');

// open modal (locks body scroll)
modal.open();

// close modal (runs exit animation and unlocks body scroll)
modal.close();

// close, await exit transition, and remove element from DOM
await modal.destroy();
```

#### events

| event   | detail | bubbles | description                  |
| :------ | :----- | :------ | :--------------------------- |
| `open`  | none   | no      | dispatched when modal opens  |
| `close` | none   | no      | dispatched when modal closes |

#### behavior

- displays an overlay dialog and prevents page scrolling while visible.
- dismisses when clicking the backdrop outside the dialog card.

---

### d-morph

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/morph.mp4" type="video/mp4">
  </video>
</p>

physics based morphing animation engine (`dMotion`) that transitions between named elements anywhere in the document with continuous geometry, border-radius, box-shadow, and form state interpolation.

```html
<d-morph name="summary-card" d-action="click" d-to="detail-card" d-duration="350" d-visible>
  <div class="card p-2">
    <h4>project summary</h4>
    <p>click to expand details</p>
  </div>
</d-morph>

<d-morph name="detail-card" d-action="click" d-to="summary-card" d-duration="350">
  <div class="card p-4">
    <h4>detailed project view</h4>
    <p>full metadata and extended controls</p>
  </div>
</d-morph>
```

#### attributes and properties

| attribute    | property | type      | default  | description                                                   |
| :----------- | :------- | :-------- | :------- | :------------------------------------------------------------ |
| `name`       | `name`   | `string`  | `""`     | unique morph identifier registered in `window.dMotion`        |
| `d-visible`  | none     | `boolean` | `false`  | marks this morph element as initially active and visible      |
| `d-action`   | none     | `string`  | `""`     | trigger actions: `"click"`, `"long-press"`, `"click-out"`     |
| `d-to`       | none     | `string`  | `""`     | target morph name to transition to                            |
| `d-duration` | none     | `number`  | computed | transition duration in ms (computed from distance if omitted) |

#### programmatic api (`window.dMotion`)

```javascript
// get a morph element instance by name
const morph = dMotion.get('summary-card');

// get the immediately previous morph element
const prev = dMotion.getPrevious();

// programmatically trigger a transition between two named elements
await dMotion.transition('summary-card', 'detail-card', optionalTriggerNode);

// register transition lifecycle hooks ('before' or 'after')
dMotion.listen('summary-card', 'detail-card', () => {
  console.log('transition starting');
}, 'before');

// unregister hook
dMotion.unlisten('summary-card', 'detail-card', callback, 'before');

// component methods on <d-morph>
morph.show();
morph.hide();
```

#### behavior

- creates smooth animated morph transitions between two elements anywhere on the page.
- preserves visual styles (size, position, border radius, shadows) and element states during animation.
- supports click, long-press, and click-out triggers.
- child elements with `d-no-morph` remain interactive without triggering the transition.

---

### d-notification

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/notification.mp4" type="video/mp4">
  </video>
</p>

toast notification component with linear progress bar countdown and automated exit lifecycle.

```html
<d-notification timer="4000" opened>
  <p>project settings saved successfully</p>
</d-notification>
```

#### attributes and properties

| attribute | property | type      | default | form | description                                                  |
| :-------- | :------- | :-------- | :------ | :--- | :----------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the toast notification is displayed                  |
| `timer`   | `timer`  | `number`  | `0`     | no   | auto dismissal duration in milliseconds (`0` disables timer) |

#### programmatic api

```javascript
const toast = select('d-notification');

// show toast and start progress bar countdown
toast.show();

// hide toast
toast.hide();

// hide, wait for animation to finish, and remove from DOM
await toast.destroy();
```

#### behavior

- displays toast notifications with an optional countdown progress bar that automatically dismisses after the timer expires.

---

### d-skeleton

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/skeleton.mp4" type="video/mp4">
  </video>
</p>

content loading placeholder shapes with customizable dimensions, line counts, and card presets.

```html
<d-skeleton type="text" lines="3" gap="0.5em"></d-skeleton>
<d-skeleton type="circle" size="3em"></d-skeleton>
<d-skeleton type="rect" width="100%" height="150px" radius="0.5em"></d-skeleton>
<d-skeleton type="card"></d-skeleton>
```

#### attributes and properties

| attribute | property | type     | default   | description                                                   |
| :-------- | :------- | :------- | :-------- | :------------------------------------------------------------ |
| `type`    | `type`   | `string` | `"text"`  | preset shape (`"text"`, `"circle"`, `"rect"`, `"card"`)       |
| `lines`   | `lines`  | `number` | `1`       | line count for `"text"` type (last line renders at 60% width) |
| `width`   | `width`  | `string` | `null`    | CSS width                                                     |
| `height`  | `height` | `string` | `null`    | CSS height                                                    |
| `size`    | `size`   | `string` | `null`    | shorthand for equal width and height on circle and rect       |
| `gap`     | `gap`    | `string` | `"0.6em"` | vertical gap between lines in text mode                       |
| `radius`  | `radius` | `string` | `null`    | custom border-radius override                                 |

---

### d-slider

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/slider.mp4" type="video/mp4">
  </video>
</p>

range slider with pointer dragging, keyboard navigation, audio feedback, and automatic indicator text binding.

```html
<d-slider id="volume" name="volume" min="0" max="1" step="0.01" value="0.75"></d-slider>
<span data-slider-volume class="text-content-l">75%</span>
```

#### attributes and properties

| attribute    | property     | type      | default                 | form | description                                             |
| :----------- | :----------- | :-------- | :---------------------- | :--- | :------------------------------------------------------ |
| `value`      | `value`      | `number`  | `0.5`                   | yes  | current numeric slider value                            |
| `min`        | `min`        | `number`  | `0`                     | no   | minimum allowable value                                 |
| `max`        | `max`        | `number`  | `1`                     | no   | maximum allowable value                                 |
| `step`       | `step`       | `number`  | `0.01`                  | no   | step increment for dragging and arrow keys              |
| `name`       | `name`       | `string`  | `""`                    | no   | form payload field name                                 |
| `id`         | `id`         | `string`  | `""`                    | no   | component id (used for `data-slider-{id}` text binding) |
| `disabled`   | `disabled`   | `boolean` | `false`                 | no   | disables slider interactions                            |
| `thumbcolor` | `thumbColor` | `string`  | `"var(--accent)"`       | no   | thumb color variable                                    |
| `fillcolor`  | `fillColor`  | `string`  | `"var(--accent-d)"`     | no   | active track fill color variable                        |
| `trackcolor` | `trackColor` | `string`  | `"var(--border-muted)"` | no   | inactive track background color variable                |
| `sfx`        | none         | `string`  | embedded                | no   | audio sound effect URL or base64 audio payload          |

#### programmatic api

```javascript
const slider = select('d-slider');

// read current value
const current = slider.getValue();

// set new value (updates visual fill, thumb position, and data-slider indicator)
slider.setValue(0.85);
```

#### events

| event    | detail        | bubbles | description                                                  |
| :------- | :------------ | :------ | :----------------------------------------------------------- |
| `input`  | numeric value | yes     | dispatched during continuous dragging and arrow key movement |
| `change` | numeric value | yes     | dispatched on pointer up or keyboard commit                  |

#### behavior

- supports smooth dragging and keyboard arrow navigation.
- automatically updates text in any element with `data-slider-{id}` with the current percentage value.
- integrates with form submission.

---

### d-text-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/input.mp4" type="video/mp4">
  </video>
</p>

enhanced input field with icon prefixes, keyboard shortcut binding, number stepper buttons, and form integration.

```html
<d-text-input
  placeholder="search records"
  icon="dstrn-search-line"
  shortcut="cmd+k"
  name="query">
</d-text-input>
```

#### attributes and properties

| attribute      | property       | type      | default  | form | description                                                 |
| :------------- | :------------- | :-------- | :------- | :--- | :---------------------------------------------------------- |
| `placeholder`  | `placeholder`  | `string`  | `""`     | no   | placeholder text                                            |
| `icon`         | `icon`         | `string`  | `""`     | no   | icon class name                                             |
| `type`         | `type`         | `string`  | `"text"` | no   | input type (`"text"`, `"password"`, `"search"`, `"number"`) |
| `autocomplete` | `autocomplete` | `boolean` | `false`  | no   | browser autocomplete flag                                   |
| `value`        | `value`        | `string`  | `""`     | yes  | current input string value                                  |
| `name`         | `name`         | `string`  | `""`     | no   | form field name                                             |
| `id`           | `id`           | `string`  | `""`     | no   | component id                                                |
| `shortcut`     | `shortcut`     | `string`  | `""`     | no   | global keyboard shortcut (e.g. `"cmd+k"`, `"ctrl+f"`)       |
| `disabled`     | `disabled`     | `boolean` | `false`  | no   | disables input                                              |
| `readonly`     | `readonly`     | `boolean` | `false`  | no   | sets input to read only mode                                |
| `min`          | `min`          | `string`  | `""`     | no   | minimum number value                                        |
| `max`          | `max`          | `string`  | `""`     | no   | maximum number value                                        |
| `step`         | `step`         | `string`  | `"1"`    | no   | step value for number controls                              |

#### programmatic api

```javascript
const input = select('d-text-input');

// update value
input.value = 'search term';

// read or modify properties
input.placeholder = 'type command';
input.disabled = false;
```

#### events

| event    | detail       | bubbles | description                      |
| :------- | :----------- | :------ | :------------------------------- |
| `change` | string value | yes     | dispatched on input modification |

#### behavior

- supports global keyboard shortcuts (e.g. `cmd+k`) that focus the field automatically across operating systems.
- displays increment and decrement stepper controls when configured with `type="number"`.

---

### d-toggle

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/toggle.mp4" type="video/mp4">
  </video>
</p>

segmented toggle control with magnetic hover indicators and animated transitions.

```html
<d-toggle name="billing_period" value="0">
  <div default>monthly billing</div>
  <div>annual billing (save 20%)</div>
</d-toggle>
```

#### attributes and properties

| attribute         | property          | type     | default                       | form | description                                       |
| :---------------- | :---------------- | :------- | :---------------------------- | :--- | :------------------------------------------------ |
| `value`           | `value`           | `number` | `0`                           | yes  | zero indexed integer of currently selected option |
| `name`            | `name`            | `string` | `""`                          | no   | form field name                                   |
| `id`              | `id`              | `string` | `""`                          | no   | component id                                      |
| `color`           | `color`           | `string` | `"var(--container-l)"`        | no   | container background color variable               |
| `activecolor`     | `activeColor`     | `string` | `"var(--accent)"`             | no   | active option highlight color                     |
| `textcolor`       | `textColor`       | `string` | `"var(--content-l)"`          | no   | inactive option text color                        |
| `textactivecolor` | `textActiveColor` | `string` | `""`                          | no   | active option text color                          |
| `bgcolor`         | `bgColor`         | `string` | `"var(--container-l)"`        | no   | base background color                             |
| `hovercolor`      | `hoverColor`      | `string` | `"rgba(255, 255, 255, 0.05)"` | no   | hover background color                            |

#### programmatic api

```javascript
const toggle = select('d-toggle');

// get currently active button element
console.log(toggle.selected);

// activate option by index (number), string index, or button element
toggle.selected = 1;

// read or set numeric value
toggle.value = 0;
```

#### events

| event    | detail                | bubbles | description                              |
| :------- | :-------------------- | :------ | :--------------------------------------- |
| `change` | selected index number | yes     | dispatched when active selection changes |

#### behavior

- renders interactive segmented options with animated hover and selection transitions.
- supports initial selection via the `default` attribute or numeric value.

---

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

<a name="immutable-base-class"></a>

### immutable base class

all custom elements and overrides extend `dComponent`. unlike UI components, `dComponent` is the immutable framework base class providing reactivity, reflection, and lifecycle management. the compiler always preserves the core `dComponent` implementation at the root of the bundle so all components inherit from it consistently.

---

<a name="dcomponent-base-class"></a>

## dComponent (base class)

`dComponent` is the authoring base class for all custom web components in dframework. it provides property reflection, reactive proxy state, form integration, surgical DOM rendering, and automatic memory cleanup.

```javascript
class dCounter extends dComponent {
  static tag = 'd-counter';
  static form = true; // opt into native form submission integration

  static props = {
    value: { type: 'number', default: 0, form: true },
    step: { type: 'number', default: 1 },
    disabled: { type: 'boolean', default: false },
    label: { type: 'string', default: 'count' }
  };

  template() {
    // called once on mount to establish initial innerHTML
    return `
      <span class="label">${this.label}</span>
      <button class="dec" type="button">−</button>
      <span class="value">${this.value}</span>
      <button class="inc" type="button">+</button>
    `;
  }

  mount() {
    // called once after template initialization
    this.effect(() => {
      // reading this.state or props registers auto dependencies
      document.title = `count: ${this.state.value ?? this.value}`;
      return () => { document.title = 'app'; }; // optional cleanup
    });
  }

  render() {
    // called automatically on prop or state changes
    // use for surgical DOM updates only (never mutate this.state here)
    this.refs('.value')[0].textContent = this.state.value ?? this.value;
    this.refs('button.dec')[0].disabled = this.disabled;
    this.refs('button.inc')[0].disabled = this.disabled;

    // listeners attached inside render() are cleared and rebound automatically
    this.listen(this.refs('button.dec')[0], 'click', () => this.decrement());
    this.listen(this.refs('button.inc')[0], 'click', () => this.increment());
  }

  onPropChanged(name, oldVal, newVal) {
    // called on attribute or property modifications
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

<a name="component-definition-and-registration"></a>

### component definition and registration

register components using `dComponent.define(Class)`:

```javascript
dComponent.define(dCounter);
```

- automatically defines getter and setter properties on the prototype with type coercion (`number`, `boolean`, `string`, `object`, `array`).
- binds observed attributes to property setters.
- registers with `customElements.define` using `Class.tag`.

<a name="lifecycle-methods"></a>

### lifecycle methods

| method                          | timing               | description                                           |
| :------------------------------ | :------------------- | :---------------------------------------------------- |
| `template()`                    | before mount         | returns initial HTML string to populate inner DOM     |
| `mount()`                       | on first connection  | initializes component logic and registers effects     |
| `connected()`                   | on every connection  | called whenever element is attached to document       |
| `render()`                      | on state/prop change | performs surgical DOM node updates                    |
| `onPropChanged(name, old, new)` | on prop mutation     | reacts to specific property changes                   |
| `destroy()`                     | on disconnection     | teardown hook called when element is removed from DOM |

<a name="rendering-strategy-and-surgical-updates"></a>

### rendering strategy and surgical updates

`dComponent` avoids virtual DOM overhead. DOM nodes are created once in `template()` or `mount()`, while dynamic updates are applied surgically inside `render()`.

> [!IMPORTANT]
> `this.state` must never be modified inside `render()`. state mutations inside `render()` are blocked to prevent infinite update loops.

<a name="reactive-state"></a>

### reactive state

```javascript
// update state and trigger microtask batched render()
this.setState({ active: true });

// functional updater
this.setState(s => { s.count++; s.updatedAt = Date.now(); });

// update state without triggering render() (still triggers effect loops)
this.setState({ timer: Date.now() }, false);
```

<a name="effects-and-dependency-tracking"></a>

### effects and dependency tracking

```javascript
// auto tracking effect (tracks any accessed this.state or prop)
this.effect(() => {
  console.log('value updated:', this.state.value);
  return () => console.log('cleaning up effect');
});

// explicit dependency array effect
this.effect(() => {
  fetchData(this.query);
}, () => [this.query]);
```

<a name="automatic-cleanup-apis"></a>

### automatic cleanup apis

these methods automatically clean up listeners, timers, and animation frame requests when the component is removed from the DOM:

```javascript
// event listener with auto cleanup
this.listen(element, 'click', (e) => this.handleClick(e));

// listen to multiple elements
this.listenAll(elements, 'click', (e) => this.handleClick(e));

// unlisten manually if needed
this.unlisten(element, 'click', handler);

// auto cleared timers
this.setTimeout(() => this.tick(), 1000);
this.setInterval(() => this.refresh(), 5000);

// auto cleared animation frame
this.requestAnimationFrame((time) => this.draw(time));
```

<a name="dom-caching-and-refs"></a>

### dom caching and refs

`this.refs(selector)` caches queried element arrays between renders to eliminate repeated DOM query overhead:

```javascript
const [inputEl] = this.refs('input.search');
```

<a name="inner-content-capture"></a>

### inner content capture

access original light DOM nodes passed into the custom element before `template()` execution:

```javascript
this.originalChildren // array of cloned child nodes
this.originalHTML     // original innerHTML string
this.cloneChildren()  // returns fresh clones of original child nodes
```

<a name="form-integration"></a>

### form integration

when `static form = true`, `dComponent` integrates with parent `<form>` elements and submits properties marked with `form: true`:

```javascript
static form = true;
static props = {
  value: { type: 'string', default: '', form: true }
};

// access parent form element
console.log(this.form);
```

supports single values, file instances (`File`), file arrays (`File[]`), and multiple form properties automatically.

<a name="internal-event-emitter"></a>

### internal event emitter

```javascript
// listen to component events
this.on('custom-event', (data) => console.log(data));

// remove listener
this.off('custom-event', handler);

// emit event to internal listeners
this.emit('custom-event', { id: 123 });
```

<a name="websocket-integration"></a>

### websocket integration

```javascript
// emit event through global websocket connection
this.wire('chat:send', { message: 'hello' });
```
