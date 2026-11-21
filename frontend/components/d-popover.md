# d-popover

top layer floating popover container. supports click, hover, and manual triggers, automatic viewport flipping, scroll tracking across nested containers, and nested popovers.

```html
<d-popover side="bottom" align="start">
  <button slot="trigger" class="btn btn-outline">open popover</button>
  <div slot="content">
    <div class="d-popover-header">
      <h4 class="d-popover-title">dimensions</h4>
      <p class="d-popover-desc">set the dimensions for the layer.</p>
    </div>
    <d-text-input placeholder="width" value="100%"></d-text-input>
  </div>
</d-popover>

<d-popover trigger="hover" side="top" align="center">
  <button slot="trigger" class="btn btn-gray">hover reveal</button>
  <div slot="content">popover content</div>
</d-popover>
```

<a name="attributes-and-properties-d-popover"></a>
## attributes and properties

| attribute                | property              | type      | default    | form | description                                                                                      |
| :----------------------- | :-------------------- | :-------- | :--------- | :--- | :----------------------------------------------------------------------------------------------- |
| `opened`                 | `opened`              | `boolean` | `false`    | no   | open state reflected on the host                                                                 |
| `trigger`                | `trigger`             | `string`  | `"click"`  | no   | activation mode (`"click"`, `"hover"`, or `"manual"`)                                            |
| `side`                   | `side`                | `string`  | `"bottom"` | no   | preferred placement (`"bottom"`, `"top"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`) |
| `align`                  | `align`               | `string`  | `"start"`  | no   | alignment along the anchor axis (`"start"`, `"center"`, or `"end"`)                              |
| `offset`                 | `offset`              | `number`  | `8`        | no   | pixel gap between anchor and popover surface                                                     |
| `match-width`            | `matchWidth`          | `boolean` | `false`    | no   | locks popover width to match its anchor element                                                  |
| `hover-delay`            | `hoverDelay`          | `number`  | `120`      | no   | hover open and close grace delay in milliseconds                                                 |
| `target`                 | `target`              | `string`  | `""`       | no   | optional CSS selector for an external anchor element                                             |
| `close-on-click-outside` | `closeOnClickOutside` | `boolean` | `true`     | no   | closes when clicking outside the anchor and popover tree                                         |
| `close-on-escape`        | `closeOnEscape`       | `boolean` | `true`     | no   | closes when pressing `Escape`                                                                    |
| `exclusive`              | `exclusive`           | `boolean` | `true`     | no   | closes unrelated open popovers on open while keeping ancestor popovers open                      |
| `unstyled`               | `unstyled`            | `boolean` | `false`    | no   | removes default surface padding, border, and background for custom overlays                      |

<a name="programmatic-api-d-popover"></a>
## programmatic api

```javascript
const pop = select('d-popover');

pop.open();
pop.close();
pop.toggle();
pop.reposition();
pop.setAnchor(select('#custom-anchor'));
const anchor = pop.getAnchor();

// close all open popovers
dPopover.closeAll();
```

<a name="events-d-popover"></a>
## events

| event   | detail | bubbles | description                        |
| :------ | :----- | :------ | :--------------------------------- |
| `open`  | none   | yes     | dispatched when the popover opens  |
| `close` | none   | yes     | dispatched when the popover closes |

<a name="behavior-d-popover"></a>
## behavior

- renders inside the browser top layer so panels are never clipped by `overflow: hidden` or stacking contexts, and stays locked to its anchor during nested container or page scrolling.
- automatically flips to the opposite side when space is constrained and animates from the resolved anchor edge with a directional spring curve.
- accepts either explicit `slot="trigger"` / `slot="content"` children, positional children (first child as trigger, remaining as content), or an external `target` anchor selector.
