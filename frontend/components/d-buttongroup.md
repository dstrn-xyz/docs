# d-buttongroup

container that fuses adjacent buttons, text addons, inputs, and popover or tooltip triggers into a unified horizontal or vertical bar with shared outer radius and clean hairline borders.

```html
<d-buttongroup>
  <button class="btn btn-outline">archive</button>
  <button class="btn btn-outline">report</button>
  <button class="btn btn-outline">snooze</button>
</d-buttongroup>

<d-buttongroup>
  <button class="btn btn-gray">copy</button>
  <span class="d-btngroup-sep"></span>
  <button class="btn btn-gray">paste</button>
</d-buttongroup>

<d-buttongroup orientation="vertical">
  <button class="btn btn-outline"><i class="dstrn-add-line"></i></button>
  <button class="btn btn-outline"><i class="dstrn-subtract-line"></i></button>
</d-buttongroup>

<d-buttongroup>
  <span class="d-btngroup-text">https://</span>
  <d-text-input placeholder="example.com"></d-text-input>
  <button class="btn btn-outline">go</button>
</d-buttongroup>
```

<a name="attributes-and-properties-d-buttongroup"></a>
## attributes and properties

| attribute     | property      | type      | default        | form | description                                                                   |
| :------------ | :------------ | :-------- | :------------- | :--- | :---------------------------------------------------------------------------- |
| `orientation` | `orientation` | `string`  | `"horizontal"` | no   | layout axis (`"horizontal"` or `"vertical"`)                                  |
| `disabled`    | `disabled`    | `boolean` | `false`        | no   | disables all interactive controls inside the group without losing prior state |
| `dividers`    | `dividers`    | `boolean` | `false`        | no   | adds hairline dividers automatically between adjacent solid buttons           |

<a name="behavior-d-buttongroup"></a>
## behavior

- collapses overlapping internal borders and rounds only outer corners using css logical radii for automatic RTL support.
- supports nested `<d-buttongroup>` containers, automatically adding toolbar spacing between sibling groups instead of fusing them.
- integrates seamlessly with `<d-popover>`, `<d-tooltip>`, `<d-text-input>`, `<d-combobox>`, `.d-btngroup-text` labels, and `.d-btngroup-sep` or `<hr>` dividers.
