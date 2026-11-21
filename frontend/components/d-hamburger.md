# d-hamburger

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

## attributes and properties

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

## slots

- `slot="header"`: custom header content in the mobile overlay.
- `slot="footer"`: custom footer content at the bottom of the overlay.

## programmatic api

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

## events

| event   | detail | bubbles | description                    |
| :------ | :----- | :------ | :----------------------------- |
| `open`  | none   | yes     | dispatched when overlay opens  |
| `close` | none   | yes     | dispatched when overlay closes |

## behavior

- automatically displays the mobile button and hides the desktop navigation when below the specified breakpoint.
- supports nested sliding subpanels using `d-submenu` attributes on link containers.
- automatically manages page scroll locking while open.
- closes automatically when clicking links, pressing Escape, or navigating between pages.