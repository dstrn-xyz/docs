# d-drawer

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

## attributes and properties

| attribute   | property    | type      | default  | form | description                                              |
| :---------- | :---------- | :-------- | :------- | :--- | :------------------------------------------------------- |
| `opened`    | `opened`    | `boolean` | `false`  | no   | whether the drawer is open                               |
| `direction` | `direction` | `string`  | `"left"` | no   | entrance edge (`"left"`, `"right"`, `"top"`, `"bottom"`) |

## programmatic api

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

## events

| event   | detail | bubbles | description                                     |
| :------ | :----- | :------ | :---------------------------------------------- |
| `open`  | none   | yes     | dispatched when the opening animation completes |
| `close` | none   | yes     | dispatched when the closing animation completes |

## behavior

- slides into view from the configured edge with multi layer wave animations.
- programmatic `open()`, `close()`, and `toggle()` methods return promises that resolve upon animation completion.
