# d-checkbox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/checkbox.mp4" type="video/mp4">
  </video>
</p>

form associated checkbox switch with synchronized label and native form submission integration.

```html
<d-checkbox text="enable notifications" checked name="notifications" value="1"></d-checkbox>
<d-checkbox text="accept terms" placement="before" name="terms"></d-checkbox>
```

## attributes and properties

| attribute   | property    | type      | default      | form | description                                                     |
| :---------- | :---------- | :-------- | :----------- | :--- | :-------------------------------------------------------------- |
| `checked`   | `checked`   | `boolean` | `false`      | yes  | checked state reflected in form submission                      |
| `text`      | `text`      | `string`  | `"checkbox"` | no   | label text displayed beside the toggle                          |
| `name`      | `name`      | `string`  | `""`         | no   | input field name used during form submission                    |
| `value`     | `value`     | `string`  | `"on"`       | no   | form submission payload sent when checked                       |
| `placement` | `placement` | `string`  | `"after"`    | no   | toggle position relative to the label (`"after"` or `"before"`) |
| `disabled`  | `disabled`  | `boolean` | `false`      | no   | disables interaction and omits the field from form payloads     |

## programmatic api

```javascript
const checkbox = select('d-checkbox');

// get or set checked state
checkbox.checked = true;

// update label text and placement
checkbox.text = 'allow telemetry';
checkbox.placement = 'before';
checkbox.disabled = false;

// update form payload name and value
checkbox.name = 'telemetry';
checkbox.value = 'accepted';
```

## events

| event    | detail                  | bubbles | description                                       |
| :------- | :---------------------- | :------ | :------------------------------------------------ |
| `change` | boolean (checked state) | yes     | dispatched whenever the user toggles the checkbox |

## behavior

- automatically synchronizes checked state, label text, and form submission values when toggled by the user or modified via javascript.
