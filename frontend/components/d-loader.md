# d-loader

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/loader.mp4" type="video/mp4">
  </video>
</p>

canvas arc spinner with automatic viewport intersection pause.

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

## attributes and properties

| attribute | property | type     | default | form | description                                                  |
| :-------- | :------- | :------- | :------ | :--- | :----------------------------------------------------------- |
| `color`   | `color`  | `string` | `""`    | no   | stroke color (defaults to `--accent` or computed text color) |

## programmatic api

```javascript
const loader = select('d-loader');

// destroy loader and disconnect observers
loader.destroy();
```

## behavior

- lightweight canvas spinner that automatically pauses animation when scrolled offscreen to conserve CPU.
- automatically adapts to container dimensions and renders sharply on high resolution screens.
