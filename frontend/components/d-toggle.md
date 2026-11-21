# d-toggle

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

## attributes and properties

| attribute         | property          | type     | default                | form | description                                           |
| :---------------- | :---------------- | :------- | :--------------------- | :--- | :---------------------------------------------------- |
| `value`           | `value`           | `number` | `0`                    | yes  | zero indexed integer of currently selected option     |
| `name`            | `name`            | `string` | `""`                   | no   | form field name                                       |
| `id`              | `id`              | `string` | `""`                   | no   | component id                                          |
| `color`           | `color`           | `string` | `"var(--container-l)"` | no   | container background color variable                   |
| `activecolor`     | `activeColor`     | `string` | `"var(--accent)"`      | no   | active option highlight color                         |
| `textcolor`       | `textColor`       | `string` | `""`                   | no   | inactive option text color (auto contrast by default) |
| `textactivecolor` | `textActiveColor` | `string` | `""`                   | no   | active option text color (auto contrast by default)   |
| `bgcolor`         | `bgColor`         | `string` | `"var(--container-l)"` | no   | base background color                                 |
| `hovercolor`      | `hoverColor`      | `string` | `""`                   | no   | hover background color (theme adaptive by default)    |

## programmatic api

```javascript
const toggle = select('d-toggle');

// get currently active button element
console.log(toggle.selected);

// activate option by index (number), string index, or button element
toggle.selected = 1;

// read or set numeric value
toggle.value = 0;
```

## events

| event    | detail                | bubbles | description                              |
| :------- | :-------------------- | :------ | :--------------------------------------- |
| `change` | selected index number | yes     | dispatched when active selection changes |

## behavior

- renders interactive segmented options with animated hover and selection transitions.
- supports initial selection via the `default` attribute or numeric value.
