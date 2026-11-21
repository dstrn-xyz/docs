# d-progress

progress bar indicator with optional header label, percentage readout, custom range bounds, and indeterminate loading shimmer.

```html
<d-progress value="33"></d-progress>
<d-progress value="56" label="upload progress" show-value></d-progress>
<d-progress indeterminate label="syncing assets"></d-progress>
```

<a name="attributes-and-properties-d-progress"></a>
## attributes and properties

| attribute       | property        | type      | default | form | description                                                   |
| :-------------- | :-------------- | :-------- | :------ | :--- | :------------------------------------------------------------ |
| `value`         | `value`         | `number`  | `0`     | no   | current progress value within `min` and `max`                 |
| `min`           | `min`           | `number`  | `0`     | no   | lower bound for ratio calculation                             |
| `max`           | `max`           | `number`  | `100`   | no   | upper bound representing 100 percent completion               |
| `indeterminate` | `indeterminate` | `boolean` | `false` | no   | switches the bar into continuous indeterminate animation mode |
| `label`         | `label`         | `string`  | `""`    | no   | optional label displayed above the track                      |
| `show-value`    | `showValue`     | `boolean` | `false` | no   | displays the rounded percentage readout above the track       |
| `color`         | `color`         | `string`  | `""`    | no   | custom fill color css value (defaults to `var(--accent)`)     |
| `size`          | `size`          | `string`  | `""`    | no   | custom track height css value (defaults to `0.25em`)          |

<a name="programmatic-api-d-progress"></a>
## programmatic api

```javascript
const progress = select('d-progress');

progress.value = 72;
console.log(progress.percentage); // 72
console.log(progress.ratio);      // 0.72
```

<a name="events-d-progress"></a>
## events

| event      | detail           | bubbles | description                                                    |
| :--------- | :--------------- | :------ | :------------------------------------------------------------- |
| `complete` | number (`value`) | yes     | dispatched once when `value` reaches `max` in determinate mode |

<a name="behavior-d-progress"></a>
## behavior

- automatically pauses indeterminate animations when scrolled offscreen and respects `prefers-reduced-motion`.
- triggers a single subtle sheen pass when transitioning into the completed state.
