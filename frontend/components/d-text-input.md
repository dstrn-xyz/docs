# d-text-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/input.mp4" type="video/mp4">
  </video>
</p>

enhanced input field with icon prefixes, keyboard shortcut binding, number stepper buttons, and form integration.

```html
<d-text-input
  placeholder="search records"
  icon="dstrn-search-line"
  shortcut="cmd+k"
  name="query">
</d-text-input>
```

## attributes and properties

| attribute      | property       | type      | default  | form | description                                                 |
| :------------- | :------------- | :-------- | :------- | :--- | :---------------------------------------------------------- |
| `placeholder`  | `placeholder`  | `string`  | `""`     | no   | placeholder text                                            |
| `icon`         | `icon`         | `string`  | `""`     | no   | icon class name                                             |
| `type`         | `type`         | `string`  | `"text"` | no   | input type (`"text"`, `"password"`, `"search"`, `"number"`) |
| `autocomplete` | `autocomplete` | `boolean` | `false`  | no   | browser autocomplete flag                                   |
| `value`        | `value`        | `string`  | `""`     | yes  | current input string value                                  |
| `name`         | `name`         | `string`  | `""`     | no   | form field name                                             |
| `id`           | `id`           | `string`  | `""`     | no   | component id                                                |
| `shortcut`     | `shortcut`     | `string`  | `""`     | no   | global keyboard shortcut (e.g. `"cmd+k"`, `"ctrl+f"`)       |
| `disabled`     | `disabled`     | `boolean` | `false`  | no   | disables input                                              |
| `readonly`     | `readonly`     | `boolean` | `false`  | no   | sets input to read only mode                                |
| `min`          | `min`          | `string`  | `""`     | no   | minimum number value                                        |
| `max`          | `max`          | `string`  | `""`     | no   | maximum number value                                        |
| `step`         | `step`         | `string`  | `"1"`    | no   | step value for number controls                              |

## programmatic api

```javascript
const input = select('d-text-input');

// update value
input.value = 'search term';

// read or modify properties
input.placeholder = 'type command';
input.disabled = false;

// programmatic focus and blur
input.focus();
input.blur();
```

## events

| event    | detail       | bubbles | description                      |
| :------- | :----------- | :------ | :------------------------------- |
| `input`  | string value | yes     | dispatched on keystroke input    |
| `change` | string value | yes     | dispatched on input modification |

## behavior

- supports global keyboard shortcuts (e.g. `cmd+k`) that focus the field automatically across operating systems.
- displays increment and decrement stepper controls when configured with `type="number"`.
