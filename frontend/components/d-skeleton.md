# d-skeleton

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

## attributes and properties

| attribute | property | type     | default   | description                                                   |
| :-------- | :------- | :------- | :-------- | :------------------------------------------------------------ |
| `type`    | `type`   | `string` | `"text"`  | preset shape (`"text"`, `"circle"`, `"rect"`, `"card"`)       |
| `lines`   | `lines`  | `number` | `1`       | line count for `"text"` type (last line renders at 60% width) |
| `width`   | `width`  | `string` | `null`    | CSS width                                                     |
| `height`  | `height` | `string` | `null`    | CSS height                                                    |
| `size`    | `size`   | `string` | `null`    | shorthand for equal width and height on circle and rect       |
| `gap`     | `gap`    | `string` | `"0.6em"` | vertical gap between lines in text mode                       |
| `radius`  | `radius` | `string` | `null`    | custom border-radius override                                 |
