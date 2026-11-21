# d-radiobutton

form associated radio button with sibling group coordination, optional description text, placement control, and keyboard arrow navigation.

```html
<d-radiobutton name="plan" value="free" text="starter" description="for personal projects" checked></d-radiobutton>
<d-radiobutton name="plan" value="pro" text="pro" description="unlimited workspaces"></d-radiobutton>
<d-radiobutton name="plan" value="team" text="team" placement="after"></d-radiobutton>
```

<a name="attributes-and-properties-d-radiobutton"></a>
## attributes and properties

| attribute     | property      | type      | default    | form | description                                                        |
| :------------ | :------------ | :-------- | :--------- | :--- | :----------------------------------------------------------------- |
| `checked`     | `checked`     | `boolean` | `false`    | yes  | checked state; checking one unchecks peers sharing the same `name` |
| `name`        | `name`        | `string`  | `""`       | no   | group and form field name                                          |
| `value`       | `value`       | `string`  | `"on"`     | no   | form submission value sent when checked                            |
| `text`        | `text`        | `string`  | `""`       | no   | primary label text (falls back to initial inner text if omitted)   |
| `description` | `description` | `string`  | `""`       | no   | optional muted secondary text displayed beneath the label          |
| `placement`   | `placement`   | `string`  | `"before"` | no   | radio position relative to label (`"before"` or `"after"`)         |
| `disabled`    | `disabled`    | `boolean` | `false`    | no   | disables interaction and excludes the field from form submission   |

<a name="programmatic-api-d-radiobutton"></a>
## programmatic api

```javascript
const proRadio = select('d-radiobutton[value="pro"]');
proRadio.checked = true;
proRadio.description = 'priority support included';
```

<a name="events-d-radiobutton"></a>
## events

| event    | detail           | bubbles | description                                        |
| :------- | :--------------- | :------ | :------------------------------------------------- |
| `change` | string (`value`) | yes     | dispatched when the user selects this radio button |

<a name="behavior-d-radiobutton"></a>
## behavior

- automatically unchecks sibling `<d-radiobutton>` elements with the same `name` within the same `<form>`, `<d-form>`, or document scope.
- supports keyboard navigation with `ArrowUp`/`ArrowLeft`, `ArrowDown`/`ArrowRight`, `Space`, and `Enter`, skipping disabled options.
- respects `prefers-reduced-motion`.
