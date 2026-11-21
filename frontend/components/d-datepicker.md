# d-datepicker

form associated date picker component combining `<d-popover>` and `<d-calendar>` with native trigger styling, clear button, single/range date modes, localized date formatting, and keyboard shortcuts.

```html
<!-- single date selection with placeholder -->
<d-datepicker name="due_date" placeholder="pick a date"></d-datepicker>

<!-- localized date display with initial value -->
<d-datepicker name="event_date" value="2026-11-19" format="localized" clearable></d-datepicker>

<!-- date range picker -->
<d-datepicker name="reservation" mode="range" value="2026-11-10,2026-11-20" clearable></d-datepicker>

<!-- min and max limits -->
<d-datepicker min="2026-11-01" max="2026-11-30"></d-datepicker>
```

<a name="attributes-and-properties-d-datepicker"></a>
## attributes and properties

| attribute           | property          | type      | default    | form | description                                                                         |
| :------------------ | :---------------- | :-------- | :--------- | :--- | :---------------------------------------------------------------------------------- |
| `name`              | `name`            | `string`  | `""`       | no   | form field name for FormData and native form submissions                            |
| `value`             | `value`           | `string`  | `""`       | yes  | selected date value (`YYYY-MM-DD` or range `from,to`)                               |
| `mode`              | `mode`            | `string`  | `"single"` | no   | selection mode (`"single"`, `"multiple"`, or `"range"`)                             |
| `placeholder`       | `placeholder`     | `string`  | `""`       | no   | trigger placeholder when empty (defaults to `"pick a date"` or `"pick date range"`) |
| `format`            | `format`          | `string`  | `""`       | no   | display text format (`"localized"` for formatted date, or empty for ISO)            |
| `era`               | `era`             | `string`  | `""`       | no   | calendar era system (`"japanese"` for Wareki display)                               |
| `min`               | `min`             | `string`  | `""`       | no   | minimum selectable date in `YYYY-MM-DD` format                                      |
| `max`               | `max`             | `string`  | `""`       | no   | maximum selectable date in `YYYY-MM-DD` format                                      |
| `locale`            | `locale`          | `string`  | `""`       | no   | BCP 47 locale code for calendar and localized date formatting                       |
| `clearable`         | `clearable`       | `boolean` | `false`    | no   | whether to display a clear button when a date is selected                           |
| `disabled`          | `disabled`        | `boolean` | `false`    | no   | disables the trigger and inner calendar                                             |
| `week-starts-on`    | `weekStartsOn`    | `number`  | `0`        | no   | first day of week passed to inner calendar (`0` Sunday or `1` Monday)               |
| `show-outside-days` | `showOutsideDays` | `boolean` | `true`     | no   | whether to show adjacent month padding days in the calendar                         |
| `close-on-select`   | `closeOnSelect`   | `boolean` | `true`     | no   | automatically closes popover after selecting a date or range                        |
| `side`              | `side`            | `string`  | `"bottom"` | no   | preferred popover placement side (`"top"` or `"bottom"`)                            |
| `align`             | `align`           | `string`  | `"center"` | no   | preferred popover alignment (`"start"`, `"center"`, or `"end"`)                     |

<a name="programmatic-api-d-datepicker"></a>
## programmatic api

```javascript
const picker = select('d-datepicker');

// control popover
picker.open();
picker.close();
picker.toggle();
picker.clear();

// state getters
picker.isOpen;        // boolean
picker.selectedDate;  // Date object or null in single mode
picker.range;         // { from: Date, to: Date } in range mode
picker.value;         // string
```

<a name="events-d-datepicker"></a>
## events

| event    | detail                             | bubbles | description                                 |
| :------- | :--------------------------------- | :------ | :------------------------------------------ |
| `change` | `{ value, from, to, date, dates }` | yes     | dispatched when date selection changes      |
| `input`  | `{ value }`                        | yes     | dispatched when value changes               |
| `open`   | none                               | yes     | dispatched when the calendar popover opens  |
| `close`  | none                               | yes     | dispatched when the calendar popover closes |

<a name="behavior-d-datepicker"></a>
## behavior

- renders an interactive input with a calendar icon, date preview or placeholder, and clear action.
- opens a floating `<d-calendar>` inside `<d-popover>`.
- in single date mode, selecting a day updates the trigger and closes the popover.
- in range mode, displays `YYYY-MM-DD - YYYY-MM-DD` and closes once both start and end endpoints are selected.
- `Enter`, `Space`, or `ArrowDown` opens the popover; `Escape` closes it; `Backspace` or `Delete` clears a clearable date selection.
