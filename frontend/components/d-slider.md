# d-slider

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

## attributes and properties

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
| `sfx`        | `sfx`        | `string`  | embedded                | no   | audio sound effect URL or base64 audio payload          |

## programmatic api

```javascript
const slider = select('d-slider');

// read current value
const current = slider.getValue();

// set new value (updates visual fill, thumb position, and data-slider indicator)
slider.setValue(0.85);
```

## events

| event    | detail        | bubbles | description                                                  |
| :------- | :------------ | :------ | :----------------------------------------------------------- |
| `input`  | numeric value | yes     | dispatched during continuous dragging and arrow key movement |
| `change` | numeric value | yes     | dispatched on pointer up or keyboard commit                  |

## behavior

- supports smooth dragging and keyboard arrow navigation.
- automatically updates text in any element with `data-slider-{id}` with the current percentage value.
- integrates with form submission.
