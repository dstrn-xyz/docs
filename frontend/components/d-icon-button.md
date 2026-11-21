# d-icon-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/icon-button.mp4" type="video/mp4">
  </video>
</p>

icon button wrapper supporting custom icons, sizes, and dynamic theme active colors.

```html
<d-icon-button icon="dstrn-heart-line" size="2em" color="var(--content-l)" activecolor="var(--red)"></d-icon-button>
```

## attributes and properties

| attribute     | property      | type     | default              | form | description                                              |
| :------------ | :------------ | :------- | :------------------- | :--- | :------------------------------------------------------- |
| `icon`        | `icon`        | `string` | `""`                 | no   | css icon class (e.g. `dstrn-heart-line`, `dstrn-search`) |
| `size`        | `size`        | `string` | `"4.2em"`            | no   | font size of the icon container                          |
| `color`       | `color`       | `string` | `"var(--content-l)"` | no   | default icon color                                       |
| `activecolor` | `activeColor` | `string` | `"var(--accent)"`    | no   | active icon color                                        |
| `id`          | `id`          | `string` | `""`                 | no   | component id                                             |
