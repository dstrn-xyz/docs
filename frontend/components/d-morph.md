# d-morph

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

## attributes and properties

| attribute    | property | type      | default  | description                                                   |
| :----------- | :------- | :-------- | :------- | :------------------------------------------------------------ |
| `name`       | `name`   | `string`  | `""`     | unique morph identifier registered in `window.dMotion`        |
| `d-visible`  | none     | `boolean` | `false`  | marks this morph element as initially active and visible      |
| `d-action`   | none     | `string`  | `""`     | trigger actions: `"click"`, `"long-press"`, `"click-out"`     |
| `d-to`       | none     | `string`  | `""`     | target morph name to transition to                            |
| `d-duration` | none     | `number`  | computed | transition duration in ms (computed from distance if omitted) |

## programmatic api (`window.dMotion`)

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

## behavior

- creates smooth animated morph transitions between two elements anywhere on the page.
- preserves visual styles (size, position, border radius, shadows) and element states during animation.
- supports click, long-press, and click-out triggers.
- child elements with `d-no-morph` remain interactive without triggering the transition.
