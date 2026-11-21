# d-color-picker

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/color-picker.mp4" type="video/mp4">
  </video>
</p>

color picker featuring 2d saturation and brightness controls, hue slider, and alpha transparency controls.

```html
<d-color-picker value="#3b82f6" name="theme_color" label="accent color"></d-color-picker>
```

## attributes and properties

| attribute | property | type      | default          | form | description                                             |
| :-------- | :------- | :-------- | :--------------- | :--- | :------------------------------------------------------ |
| `value`   | `value`  | `string`  | `"#d3ac5f"`      | yes  | current hex color value (supports 3, 4, 6, 8 digit hex) |
| `name`    | `name`   | `string`  | `""`             | no   | input name attribute for form submission                |
| `id`      | `id`     | `string`  | `""`             | no   | element identifier                                      |
| `label`   | `label`  | `string`  | `"color picker"` | no   | header label displayed above the picker                 |
| `opened`  | `opened` | `boolean` | `false`          | no   | whether the popup picker panel is initially open        |

## programmatic api

```javascript
const picker = select('d-color-picker');

// open, close, or toggle popup panel visibility
picker.open();
picker.close();
picker.toggle();

// programmatically set hex color
picker.setHex('#ef4444');

// read color state
console.log(picker.hex); // current hex string
console.log(picker.h, picker.s, picker.b, picker.a); // hsb and alpha values
```

## events

| event    | detail     | bubbles | description                                              |
| :------- | :--------- | :------ | :------------------------------------------------------- |
| `change` | hex string | yes     | dispatched when color changes via drag, input, or setHex |
| `input`  | hex string | yes     | dispatched continuously during dragging                  |

## behavior

- interactive 2D color plane, hue bar, and alpha slider with checkered transparency.
- mounts its floating picker panel through an internal `d-popover` top layer, flips automatically near viewport edges, stays aligned on scroll, and closes on outside click or `Escape`.
- supports dragging even outside the component bounds.
- automatically redraws on element resize and renders sharply on high resolution displays.
- clicking the reset icon restores the initial color value.
