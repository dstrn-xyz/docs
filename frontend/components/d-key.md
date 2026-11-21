# d-key

tactile keyboard keycap badge and optional form associated shortcut recorder input. automatically splits chord strings into formatted keycap pills, adapts modifier symbols to the user's platform, and supports live key highlighting and interactive recording.

```html
<d-key>cmd+k</d-key>
<d-key separator="">cmd,shift,p</d-key>
<d-key live>cmd+s</d-key>
<d-key editable name="shortcut" value="cmd+b"></d-key>

<span class="d-key-group">
  <d-key>cmd+b</d-key>
  <d-key>cmd+k</d-key>
</span>
```

<a name="attributes-and-properties-d-key"></a>
## attributes and properties

| attribute     | property      | type      | default       | form | description                                                                  |
| :------------ | :------------ | :-------- | :------------ | :--- | :--------------------------------------------------------------------------- |
| `value`       | `value`       | `string`  | `""`          | yes  | normalized chord string (e.g. `"cmd+k"`), readable even when seeded via text |
| `name`        | `name`        | `string`  | `""`          | no   | form field name used when submitted inside a form                            |
| `separator`   | `separator`   | `string`  | `"+"`         | no   | separator rendered between keycaps (set `""` for adjacent pills)             |
| `editable`    | `editable`    | `boolean` | `false`       | no   | turns the component into a click to record keyboard shortcut input           |
| `live`        | `live`        | `boolean` | `false`       | no   | animates matching keycaps while those physical keys are pressed              |
| `placeholder` | `placeholder` | `string`  | `"press key"` | no   | placeholder shown in editable mode when empty                                |
| `disabled`    | `disabled`    | `boolean` | `false`       | no   | disables recording and excludes the field from form submission               |
| `clearable`   | `clearable`   | `boolean` | `true`        | no   | allows clearing the recorded shortcut with `Backspace` or `Delete`           |

<a name="programmatic-api-d-key"></a>
## programmatic api

```javascript
const key = select('d-key');

// read normalized chord string or parsed token array
console.log(key.value); // "cmd+k"
console.log(key.keys);  // ["cmd", "k"]

// update or clear programmatically
key.value = 'ctrl+shift+p';
key.clear();
```

<a name="events-d-key"></a>
## events

| event    | detail                    | bubbles | description                                                         |
| :------- | :------------------------ | :------ | :------------------------------------------------------------------ |
| `change` | string (normalized chord) | yes     | dispatched when a new shortcut is recorded or cleared in input mode |

<a name="behavior-d-key"></a>
## behavior

- parses `+` or `,` separated chords from either the `value` attribute or initial text content and always exposes the normalized string on `.value` and token list on `.keys`.
- renders platform aware glyphs (`cmd` and `mod` display `⌘` on mac and `ctrl` elsewhere, `shift` displays `⇧`, `alt` displays `⌥` on mac, `enter` displays `⏎`).
- in `editable` mode, clicking or focusing starts recording; modifiers preview live, completing with a key commits and emits `change`, `Escape` cancels, and `Backspace`/`Delete` clears when `clearable` is enabled.
- integrates inside `.btn` elements and groups with `.d-key-group`.
