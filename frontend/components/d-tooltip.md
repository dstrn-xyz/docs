# d-tooltip

popup that displays helpful information when an element receives hover or keyboard focus, built upon the top layer `d-popover` positioning engine with integrated keyboard shortcut glyphs and disabled trigger wrapping.

```html
<d-tooltip text="save document" shortcut="cmd+s" side="top">
  <button class="btn btn-outline">save</button>
</d-tooltip>

<d-tooltip text="action unavailable" side="right">
  <button class="btn btn-outline" disabled>disabled action</button>
</d-tooltip>

<d-tooltip>
  <button slot="trigger" class="btn btn-gray"><i class="dstrn-add-line"></i></button>
  <span slot="content">custom tooltip markup</span>
</d-tooltip>
```

<a name="attributes-and-properties-d-tooltip"></a>
## attributes and properties

| attribute  | property   | type      | default    | form | description                                                                                           |
| :--------- | :--------- | :-------- | :--------- | :--- | :---------------------------------------------------------------------------------------------------- |
| `text`     | `text`     | `string`  | `""`       | no   | tooltip text content                                                                                  |
| `shortcut` | `shortcut` | `string`  | `""`       | no   | keyboard shortcut combo (e.g. `"cmd+s"`, `"alt+left"`) rendered automatically via `<d-key>`           |
| `side`     | `side`     | `string`  | `"top"`    | no   | preferred placement side (`"top"`, `"bottom"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`) |
| `align`    | `align`    | `string`  | `"center"` | no   | alignment along trigger axis (`"start"`, `"center"`, `"end"`)                                         |
| `delay`    | `delay`    | `number`  | `150`      | no   | hover grace delay in milliseconds before reveal                                                       |
| `offset`   | `offset`   | `number`  | `6`        | no   | pixel distance between trigger and tooltip bubble                                                     |
| `disabled` | `disabled` | `boolean` | `false`    | no   | disables tooltip display                                                                              |

<a name="programmatic-api-d-tooltip"></a>
## programmatic api

```javascript
const tip = select('d-tooltip');

tip.open();
tip.close();
tip.toggle();
tip.setAnchor(select('#custom-anchor'));
const anchor = tip.getAnchor();
tip.text = 'new label';
tip.disabled = true;

// close all active tooltips
dTooltip.closeAll();
```

<a name="events-d-tooltip"></a>
## events

| event   | detail | bubbles | description                         |
| :------ | :----- | :------ | :---------------------------------- |
| `open`  | none   | yes     | dispatched when the tooltip reveals |
| `close` | none   | yes     | dispatched when the tooltip closes  |

<a name="behavior-d-tooltip"></a>
## behavior

- renders in the browser top layer via `d-popover`, remaining immune to `overflow: hidden` parent clipping and `z-index` stacking context traps.
- automatically displays formatted platform aware `<d-key>` badges when `shortcut` is provided.
- displays tooltips over disabled buttons by routing pointer events through its outer wrapper.
- integrates with keyboard navigation, revealing on `focusin` and hiding on `focusout`.
