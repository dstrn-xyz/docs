# d-calendar

interactive calendar featuring single, range, and multiple date selection, native month and weekday localization, carousel month transitions, and keyboard navigation.

```html
<!-- single date selection -->
<d-calendar mode="single" value="2026-11-19"></d-calendar>

<!-- date range selection with continuous track highlight -->
<d-calendar mode="range" value="2026-11-10,2026-11-20"></d-calendar>

<!-- multiple date selection -->
<d-calendar mode="multiple" value="2026-11-05,2026-11-12,2026-11-19"></d-calendar>

<!-- min and max bounds -->
<d-calendar min="2026-11-01" max="2026-11-30"></d-calendar>
```

<a name="attributes-and-properties-d-calendar"></a>
## attributes and properties

| attribute           | property          | type      | default    | form | description                                                              |
| :------------------ | :---------------- | :-------- | :--------- | :--- | :----------------------------------------------------------------------- |
| `mode`              | `mode`            | `string`  | `"single"` | no   | selection mode (`"single"`, `"range"`, or `"multiple"`)                  |
| `value`             | `value`           | `string`  | `""`       | no   | selected date (`YYYY-MM-DD`, range `from,to`, or comma-separated list)   |
| `month`             | `month`           | `number`  | `null`     | no   | active month index (`1` to `12`)                                         |
| `year`              | `year`            | `number`  | `null`     | no   | active year (e.g. `2026`)                                                |
| `min`               | `min`             | `string`  | `""`       | no   | minimum selectable date in `YYYY-MM-DD` ISO format                       |
| `max`               | `max`             | `string`  | `""`       | no   | maximum selectable date in `YYYY-MM-DD` ISO format                       |
| `locale`            | `locale`          | `string`  | `""`       | no   | BCP 47 locale code (defaults to document or navigator language)          |
| `era`               | `era`             | `string`  | `""`       | no   | calendar era system (`"japanese"` for Wareki display)                    |
| `week-starts-on`    | `weekStartsOn`    | `number`  | `0`        | no   | first day of week (`0` Sunday or `1` Monday)                             |
| `show-outside-days` | `showOutsideDays` | `boolean` | `true`     | no   | whether to show adjacent month padding days in the grid                  |
| `disabled`          | `disabled`        | `boolean` | `false`    | no   | disables interaction and dims grid opacity                               |

<a name="programmatic-api-d-calendar"></a>
## programmatic api

```javascript
const cal = select('d-calendar');

// navigate months
cal.prevMonth();
cal.nextMonth();

// getters
cal.viewMode;      // "days", "months", or "years"
cal.selectedDate;  // Date object or null in single mode
cal.range;         // { from: Date, to: Date } in range mode
cal.selectedDates; // array of Date objects in multiple mode
```

<a name="events-d-calendar"></a>
## events

| event          | detail                                    | bubbles | description                                       |
| :------------- | :---------------------------------------- | :------ | :------------------------------------------------ |
| `select`       | `{ value, date }`                         | yes     | dispatched when any day cell is clicked           |
| `change`       | `{ value, date }` (single), `{ value, from, to, fromDate, toDate }` (range), or `{ value, dates }` (multiple) | yes     | dispatched when the selection state updates       |
| `month-change` | `{ year, month, monthString, value }`     | yes     | dispatched when navigating to a new month or year |

<a name="behavior-d-calendar"></a>
## behavior

- month navigation animates with a directional carousel slide along the direction of motion.
- clicking month or year in the header zooms out to a 12 item clickable grid (12 months or 12 years in a decade) for fast jumping.
- in range mode, dragging or clicking start and end points renders a continuous highlight track spanning start, middle, and end days.
- weekday headers are rendered as localized compact 2 letter labels.
- full keyboard navigation (`ArrowLeft` / `ArrowRight` by 1 day, `ArrowUp` / `ArrowDown` by 1 week, `PageUp` / `PageDown` by month, `Enter` / `Space` to select).
- automatically bounds selectable dates within `min` and `max` limits.
