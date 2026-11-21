# d-hold-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/hold-button.mp4" type="video/mp4">
  </video>
</p>

press and hold button that prevents accidental triggers of destructive operations by requiring a continuous pointer press.

```html
<d-hold-button delay="1500" class="btn btn-red" type="submit">
  hold to delete database
</d-hold-button>
```

## attributes and properties

| attribute  | property   | type      | default    | form | description                                                     |
| :--------- | :--------- | :-------- | :--------- | :--- | :-------------------------------------------------------------- |
| `delay`    | `delay`    | `number`  | `800`      | no   | hold duration in milliseconds required before triggering action |
| `type`     | `type`     | `string`  | `"button"` | no   | button type (`"button"`, `"submit"`)                            |
| `disabled` | `disabled` | `boolean` | `false`    | no   | whether the button is disabled                                  |

## programmatic api

```javascript
const holdBtn = select('d-hold-button');

// adjust required hold duration in ms
holdBtn.delay = 2000;

// toggle disabled state
holdBtn.disabled = true;
```

## events

| event   | detail             | bubbles | description                                               |
| :------ | :----------------- | :------ | :-------------------------------------------------------- |
| `click` | native click event | yes     | dispatched only after holding for the full delay duration |

## behavior

- requires continuous press for the configured duration before triggering the click action.
- releasing before completion cancels the action and resets visual progress.
- automatically submits the surrounding form when configured with `type="submit"`.
