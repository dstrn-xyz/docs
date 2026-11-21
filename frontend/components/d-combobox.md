# d-combobox

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/combobox.mp4" type="video/mp4">
  </video>
</p>

searchable, extensible select combobox with automatic floating placement, keyboard navigation, and dynamic option injection.

```html
<d-combobox
  placeholder="select department"
  allow-search
  allow-input
  name="department">
  <option value="eng" selected>engineering</option>
  <option value="des">product design</option>
  <option value="ops">operations</option>
</d-combobox>
```

## attributes and properties

| attribute      | property       | type      | default        | form | description                                                                         |
| :------------- | :------------- | :-------- | :------------- | :--- | :---------------------------------------------------------------------------------- |
| `placeholder`  | `placeholder`  | `string`  | `""`           | no   | placeholder text displayed when no option is selected                               |
| `value`        | `value`        | `string`  | `null`         | yes  | currently selected option value                                                     |
| `allow-search` | `allowSearch`  | `boolean` | `false`        | no   | enables real time search filtering input                                            |
| `allow-input`  | `allowInput`   | `boolean` | `false`        | no   | allows adding custom options via input and plus button                              |
| `horizontal`   | `isHorizontal` | `boolean` | `false`        | no   | switches option layout from vertical list to horizontal chips                       |
| `empty-text`   | `emptyText`    | `string`  | `"no results"` | no   | custom empty state message displayed when no options match                          |
| `name`         | `name`         | `string`  | `""`           | no   | form field name                                                                     |
| `id`           | `id`           | `string`  | `""`           | no   | component id                                                                        |
| `options`      | `options`      | `array`   | `[]`           | no   | array of `{value, text, content, html}` objects (set `html: true` for rich content) |

## programmatic api

```javascript
const combo = select('d-combobox');

// add option dynamically
combo.addOption('mktg', 'marketing');

// remove option by value
combo.removeOption('ops');

// clear all options
combo.clearOptions();

// set selected value programmatically
combo.value = 'eng';

// close all open comboboxes across document
dCombobox.closeAll();
```

## events

| event    | detail                | bubbles | description                                    |
| :------- | :-------------------- | :------ | :--------------------------------------------- |
| `change` | selected value string | yes     | dispatched when an option is selected or added |
| `open`   | none                  | yes     | dispatched when the dropdown panel opens       |
| `close`  | none                  | yes     | dispatched when the dropdown panel closes      |

## behavior

- automatically parses child `<option>` tags declared in HTML.
- automatically calculates viewport space to open above or below without overflowing the screen.
- keeps floating coordinates aligned during scrolling and resizing.
- provides full keyboard navigation (arrow keys, home, end, enter, escape).
- displays an empty state indicator when search queries return no matches.
