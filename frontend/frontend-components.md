# frontend components

- [frontend components](#frontend-components)
  - [introduction](#introduction)
  - [built in components](#built-in-components)
    - [d-breadcrumb](#d-breadcrumb)
      - [attributes and properties](#attributes-and-properties-d-breadcrumb)
      - [programmatic api](#programmatic-api-d-breadcrumb)
      - [events](#events-d-breadcrumb)
      - [behavior](#behavior-d-breadcrumb)
    - [d-buttongroup](#d-buttongroup)
      - [attributes and properties](#attributes-and-properties-d-buttongroup)
      - [behavior](#behavior-d-buttongroup)
    - [d-calendar](#d-calendar)
      - [attributes and properties](#attributes-and-properties-d-calendar)
      - [programmatic api](#programmatic-api-d-calendar)
      - [events](#events-d-calendar)
      - [behavior](#behavior-d-calendar)
    - [d-datepicker](#d-datepicker)
      - [attributes and properties](#attributes-and-properties-d-datepicker)
      - [programmatic api](#programmatic-api-d-datepicker)
      - [events](#events-d-datepicker)
      - [behavior](#behavior-d-datepicker)
    - [d-checkbox](#d-checkbox)
      - [attributes and properties](#attributes-and-properties)
      - [programmatic api](#programmatic-api)
      - [events](#events)
      - [behavior](#behavior)
    - [d-codeblock](#d-codeblock)
      - [attributes and properties](#attributes-and-properties-d-codeblock)
      - [programmatic api](#programmatic-api-d-codeblock)
      - [events](#events-d-codeblock)
      - [behavior](#behavior-d-codeblock)
    - [d-color-picker](#d-color-picker)
      - [attributes and properties](#attributes-and-properties-1)
      - [programmatic api](#programmatic-api-1)
      - [events](#events-1)
      - [behavior](#behavior-1)
    - [d-combobox](#d-combobox)
      - [attributes and properties](#attributes-and-properties-2)
      - [programmatic api](#programmatic-api-2)
      - [events](#events-2)
      - [behavior](#behavior-2)
    - [d-context-menu](#d-context-menu)
      - [attributes and properties](#attributes-and-properties-3)
      - [programmatic api](#programmatic-api-3)
      - [events](#events-3)
      - [behavior](#behavior-3)
    - [d-drawer](#d-drawer)
      - [attributes and properties](#attributes-and-properties-4)
      - [programmatic api](#programmatic-api-4)
      - [events](#events-4)
      - [behavior](#behavior-4)
    - [d-dropdown](#d-dropdown)
      - [attributes and properties](#attributes-and-properties-5)
      - [programmatic api](#programmatic-api-5)
      - [behavior](#behavior-5)
    - [d-file-input](#d-file-input)
      - [attributes and properties](#attributes-and-properties-6)
      - [programmatic api](#programmatic-api-6)
      - [events](#events-5)
      - [behavior](#behavior-6)
    - [d-form](#d-form)
      - [attributes and properties](#attributes-and-properties-d-form)
      - [programmatic api](#programmatic-api-d-form)
      - [events](#events-d-form)
      - [behavior](#behavior-d-form)
      - [response lifecycle](#response-lifecycle)
    - [d-hamburger](#d-hamburger)
      - [attributes and properties](#attributes-and-properties-7)
      - [slots](#slots)
      - [programmatic api](#programmatic-api-7)
      - [events](#events-6)
      - [behavior](#behavior-7)
    - [d-hold-button](#d-hold-button)
      - [attributes and properties](#attributes-and-properties-8)
      - [programmatic api](#programmatic-api-8)
      - [events](#events-7)
      - [behavior](#behavior-8)
    - [d-icon-button](#d-icon-button)
      - [attributes and properties](#attributes-and-properties-9)
    - [d-image-input](#d-image-input)
      - [attributes and properties](#attributes-and-properties-10)
      - [programmatic api](#programmatic-api-9)
      - [events](#events-8)
      - [behavior](#behavior-9)
    - [d-key](#d-key)
      - [attributes and properties](#attributes-and-properties-d-key)
      - [programmatic api](#programmatic-api-d-key)
      - [events](#events-d-key)
      - [behavior](#behavior-d-key)
    - [d-loader](#d-loader)
      - [attributes and properties](#attributes-and-properties-11)
      - [programmatic api](#programmatic-api-10)
      - [behavior](#behavior-10)
    - [d-link](#d-link)
      - [attributes and properties](#attributes-and-properties-d-link)
      - [programmatic api](#programmatic-api-d-link)
      - [behavior](#behavior-d-link)
    - [d-menubar](#d-menubar)
      - [attributes and properties](#attributes-and-properties-d-menubar)
      - [programmatic api](#programmatic-api-d-menubar)
      - [events](#events-d-menubar)
      - [behavior](#behavior-d-menubar)
    - [d-modal](#d-modal)
      - [attributes and properties](#attributes-and-properties-12)
      - [programmatic api](#programmatic-api-11)
      - [events](#events-9)
      - [behavior and styling boundaries](#behavior-and-styling-boundaries)
    - [d-morph](#d-morph)
      - [attributes and properties](#attributes-and-properties-13)
      - [programmatic api (`window.dMotion`)](#programmatic-api-windowdmotion)
      - [behavior](#behavior-11)
    - [d-navmenu](#d-navmenu)
      - [attributes and properties](#attributes-and-properties-d-navmenu)
      - [programmatic api](#programmatic-api-d-navmenu)
      - [events](#events-d-navmenu)
      - [behavior and d-hamburger integration](#behavior-and-d-hamburger-integration)
    - [d-notification](#d-notification)
      - [attributes and properties](#attributes-and-properties-14)
      - [programmatic api](#programmatic-api-12)
      - [behavior](#behavior-12)
    - [d-otp](#d-otp)
      - [attributes and properties](#attributes-and-properties-d-otp)
      - [programmatic api](#programmatic-api-d-otp)
      - [events](#events-d-otp)
      - [behavior](#behavior-d-otp)
    - [d-popover](#d-popover)
      - [attributes and properties](#attributes-and-properties-d-popover)
      - [programmatic api](#programmatic-api-d-popover)
      - [events](#events-d-popover)
      - [behavior](#behavior-d-popover)
    - [d-progress](#d-progress)
      - [attributes and properties](#attributes-and-properties-d-progress)
      - [programmatic api](#programmatic-api-d-progress)
      - [events](#events-d-progress)
      - [behavior](#behavior-d-progress)
    - [d-radiobutton](#d-radiobutton)
      - [attributes and properties](#attributes-and-properties-d-radiobutton)
      - [programmatic api](#programmatic-api-d-radiobutton)
      - [events](#events-d-radiobutton)
      - [behavior](#behavior-d-radiobutton)
    - [d-questionnaire](#d-questionnaire)
      - [attributes and properties](#attributes-and-properties-d-questionnaire)
      - [programmatic api](#programmatic-api-d-questionnaire)
      - [events](#events-d-questionnaire)
      - [behavior](#behavior-d-questionnaire)
    - [d-sidebar](#d-sidebar)
      - [attributes and properties](#attributes-and-properties-d-sidebar)
      - [programmatic api](#programmatic-api-d-sidebar)
      - [events](#events-d-sidebar)
      - [behavior](#behavior-d-sidebar)
    - [d-skeleton](#d-skeleton)
      - [attributes and properties](#attributes-and-properties-15)
    - [d-slider](#d-slider)
      - [attributes and properties](#attributes-and-properties-16)
      - [programmatic api](#programmatic-api-13)
      - [events](#events-10)
      - [behavior](#behavior-13)
    - [d-text-input](#d-text-input)
      - [attributes and properties](#attributes-and-properties-17)
      - [programmatic api](#programmatic-api-14)
      - [events](#events-11)
      - [behavior](#behavior-14)
    - [d-toggle](#d-toggle)
      - [attributes and properties](#attributes-and-properties-18)
      - [programmatic api](#programmatic-api-15)
      - [events](#events-12)
      - [behavior](#behavior-15)
    - [d-tooltip](#d-tooltip)
      - [attributes and properties](#attributes-and-properties-d-tooltip)
      - [programmatic api](#programmatic-api-d-tooltip)
      - [events](#events-d-tooltip)
      - [behavior](#behavior-d-tooltip)
    - [d-virtual-list](#d-virtual-list)
      - [attributes and properties](#attributes-and-properties-d-virtual-list)
      - [programmatic api](#programmatic-api-d-virtual-list)
      - [events](#events-d-virtual-list)
      - [behavior](#behavior-d-virtual-list)
  - [overriding built in components](#overriding-built-in-components)
    - [immutable base class](#immutable-base-class)
  - [dComponent (base class)](#dcomponent-base-class)
    - [component definition and registration](#component-definition-and-registration)
    - [lifecycle methods](#lifecycle-methods)
    - [fragment caching and batch mounting](#fragment-caching-and-batch-mounting)
    - [rendering strategy and surgical updates](#rendering-strategy-and-surgical-updates)
    - [batched microtask update queue](#batched-microtask-update-queue)
    - [reactive state](#reactive-state)
    - [effects and dependency tracking](#effects-and-dependency-tracking)
    - [automatic cleanup apis](#automatic-cleanup-apis)
    - [dom caching and refs](#dom-caching-and-refs)
    - [inner content capture](#inner-content-capture)
    - [form integration](#form-integration)
    - [internal event emitter](#internal-event-emitter)
    - [websocket integration](#websocket-integration)

<a name="introduction"></a>

## introduction

dframework ships a complete library of zero dependency, high performance custom web components compiled into the frontend bundle. components are standard custom elements with no virtual dom, no compiler requirements, and no client build step. the framework compiles and bundles component files in `public/js/components/` automatically on server boot.

<a name="built-in-components"></a>

## built in components

### d-breadcrumb

hierarchical navigation trail supporting declarative link/element children, custom text or icon separators, programmatic `items` arrays, and interactive middle item collapsing.

```html
<d-breadcrumb>
  <a d-link href="/">home</a>
  <a d-link href="/components">components</a>
  <span>breadcrumb</span>
</d-breadcrumb>

<d-breadcrumb separator="/" max-items="3">
  <a d-link href="/">home</a>
  <a d-link href="/workspace">workspace</a>
  <a d-link href="/projects">projects</a>
  <span>settings</span>
</d-breadcrumb>
```

<a name="attributes-and-properties-d-breadcrumb"></a>
#### attributes and properties

| attribute   | property    | type     | default | form | description                                                                         |
| :---------- | :---------- | :------- | :------ | :--- | :---------------------------------------------------------------------------------- |
| `separator` | `separator` | `string` | `""`    | no   | custom text (e.g. `"/"`) or icon class (`"dstrn-right-line"`); defaults to chevron  |
| `max-items` | `maxItems`  | `number` | `0`     | no   | collapses middle items into an interactive ellipsis button when count exceeds limit |
| `items`     | `items`     | `array`  | `null`  | no   | optional array of `{ label, href, link }` objects overriding light DOM children     |

<a name="programmatic-api-d-breadcrumb"></a>
#### programmatic api

```javascript
const trail = select('d-breadcrumb');

trail.expand();
trail.collapse();
trail.items = [
  { label: 'home', href: '/' },
  { label: 'docs', href: '/docs' },
  { label: 'breadcrumb' }
];
```

<a name="events-d-breadcrumb"></a>
#### events

| event      | detail | bubbles | description                                           |
| :--------- | :----- | :------ | :---------------------------------------------------- |
| `expand`   | none   | yes     | dispatched when collapsed middle items are expanded   |
| `collapse` | none   | yes     | dispatched when `collapse()` restores collapsed state |

<a name="behavior-d-breadcrumb"></a>
#### behavior

- preserves live child DOM nodes (`<a d-link>`, `<d-link>`, buttons, or nested popover triggers) without reparsing html.
- automatically highlights the final breadcrumb item as the active page and mirrors directional separator icons in RTL layouts.
- clicking the collapsed ellipsis button opens a floating popover containing the hidden middle links.

---

### d-buttongroup

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
#### attributes and properties

| attribute     | property      | type      | default        | form | description                                                                   |
| :------------ | :------------ | :-------- | :------------- | :--- | :---------------------------------------------------------------------------- |
| `orientation` | `orientation` | `string`  | `"horizontal"` | no   | layout axis (`"horizontal"` or `"vertical"`)                                  |
| `disabled`    | `disabled`    | `boolean` | `false`        | no   | disables all interactive controls inside the group without losing prior state |
| `dividers`    | `dividers`    | `boolean` | `false`        | no   | adds hairline dividers automatically between adjacent solid buttons           |

<a name="behavior-d-buttongroup"></a>
#### behavior

- collapses overlapping internal borders and rounds only outer corners using css logical radii for automatic RTL support.
- supports nested `<d-buttongroup>` containers, automatically adding toolbar spacing between sibling groups instead of fusing them.
- integrates seamlessly with `<d-popover>`, `<d-tooltip>`, `<d-text-input>`, `<d-combobox>`, `.d-btngroup-text` labels, and `.d-btngroup-sep` or `<hr>` dividers.

---

### d-calendar

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
#### attributes and properties

| attribute           | property          | type      | default    | form | description                                                              |
| :------------------ | :---------------- | :-------- | :--------- | :--- | :----------------------------------------------------------------------- |
| `mode`              | `mode`            | `string`  | `"single"` | no   | selection mode (`"single"`, `"range"`, or `"multiple"`)                  |
| `value`             | `value`           | `string`  | `""`       | no   | selected date (`YYYY-MM-DD`, range `from,to`, or comma-separated list)   |
| `month`             | `month`           | `number`  | `null`     | no   | active month index (`1` to `12`)                                         |
| `year`              | `year`            | `number`  | `null`     | no   | active year (e.g. `2026`)                                                |
| `min`               | `min`             | `string`  | `""`       | no   | minimum selectable date in `YYYY-MM-DD` ISO format                       |
| `max`               | `max`             | `string`  | `""`       | no   | maximum selectable date in `YYYY-MM-DD` ISO format                       |
| `locale`            | `locale`          | `string`  | `""`       | no   | BCP 47 locale code (defaults to document or navigator language)          |
| `week-starts-on`    | `weekStartsOn`    | `number`  | `0`        | no   | first day of week (`0` Sunday or `1` Monday)                             |
| `show-outside-days` | `showOutsideDays` | `boolean` | `true`     | no   | whether to show adjacent month padding days in the grid                  |
| `disabled`          | `disabled`        | `boolean` | `false`    | no   | disables interaction and dims grid opacity                               |

<a name="programmatic-api-d-calendar"></a>
#### programmatic api

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
#### events

| event          | detail                                    | bubbles | description                                       |
| :------------- | :---------------------------------------- | :------ | :------------------------------------------------ |
| `select`       | `{ value, date }`                         | yes     | dispatched when any day cell is clicked           |
| `change`       | `{ value, date }` (single), `{ value, from, to, fromDate, toDate }` (range), or `{ value, dates }` (multiple) | yes     | dispatched when the selection state updates       |
| `month-change` | `{ year, month, monthString, value }`     | yes     | dispatched when navigating to a new month or year |

<a name="behavior-d-calendar"></a>
#### behavior

- month navigation animates with a directional carousel slide along the direction of motion.
- clicking month or year in the header zooms out to a 12 item clickable grid (12 months or 12 years in a decade) for fast jumping.
- in range mode, dragging or clicking start and end points renders a continuous highlight track spanning start, middle, and end days.
- weekday headers are rendered as localized compact 2 letter labels.
- full keyboard navigation (`ArrowLeft` / `ArrowRight` by 1 day, `ArrowUp` / `ArrowDown` by 1 week, `PageUp` / `PageDown` by month, `Enter` / `Space` to select).
- automatically bounds selectable dates within `min` and `max` limits.

---

### d-datepicker

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
#### attributes and properties

| attribute           | property          | type      | default    | form | description                                                                         |
| :------------------ | :---------------- | :-------- | :--------- | :--- | :---------------------------------------------------------------------------------- |
| `name`              | `name`            | `string`  | `""`       | no   | form field name for FormData and native form submissions                            |
| `value`             | `value`           | `string`  | `""`       | yes  | selected date value (`YYYY-MM-DD` or range `from,to`)                               |
| `mode`              | `mode`            | `string`  | `"single"` | no   | selection mode (`"single"`, `"multiple"`, or `"range"`)                             |
| `placeholder`       | `placeholder`     | `string`  | `""`       | no   | trigger placeholder when empty (defaults to `"pick a date"` or `"pick date range"`) |
| `format`            | `format`          | `string`  | `""`       | no   | display text format (`"localized"` for formatted date, or empty for ISO)            |
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
#### programmatic api

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
#### events

| event    | detail                             | bubbles | description                                 |
| :------- | :--------------------------------- | :------ | :------------------------------------------ |
| `change` | `{ value, from, to, date, dates }` | yes     | dispatched when date selection changes      |
| `input`  | `{ value }`                        | yes     | dispatched when value changes               |
| `open`   | none                               | yes     | dispatched when the calendar popover opens  |
| `close`  | none                               | yes     | dispatched when the calendar popover closes |

<a name="behavior-d-datepicker"></a>
#### behavior

- renders an interactive input with a calendar icon, date preview or placeholder, and clear action.
- opens a floating `<d-calendar>` inside `<d-popover>`.
- in single date mode, selecting a day updates the trigger and closes the popover.
- in range mode, displays `YYYY-MM-DD - YYYY-MM-DD` and closes once both start and end endpoints are selected.
- `Enter`, `Space`, or `ArrowDown` opens the popover; `Escape` closes it; `Backspace` or `Delete` clears a clearable date selection.

---

### d-checkbox

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

#### attributes and properties

| attribute   | property    | type      | default      | form | description                                                     |
| :---------- | :---------- | :-------- | :----------- | :--- | :-------------------------------------------------------------- |
| `checked`   | `checked`   | `boolean` | `false`      | yes  | checked state reflected in form submission                      |
| `text`      | `text`      | `string`  | `"checkbox"` | no   | label text displayed beside the toggle                          |
| `name`      | `name`      | `string`  | `""`         | no   | input field name used during form submission                    |
| `value`     | `value`     | `string`  | `"on"`       | no   | form submission payload sent when checked                       |
| `placement` | `placement` | `string`  | `"after"`    | no   | toggle position relative to the label (`"after"` or `"before"`) |
| `disabled`  | `disabled`  | `boolean` | `false`      | no   | disables interaction and omits the field from form payloads     |

#### programmatic api

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

#### events

| event    | detail                  | bubbles | description                                       |
| :------- | :---------------------- | :------ | :------------------------------------------------ |
| `change` | boolean (checked state) | yes     | dispatched whenever the user toggles the checkbox |

#### behavior

- automatically synchronizes checked state, label text, and form submission values when toggled by the user or modified via javascript.

---

### d-codeblock

syntax highlighted code block component with a copy to clipboard button, filename header bar, language badge, line numbers, line highlighting, and diff mode.

```html
<!-- standard code block with title and language -->
<d-codeblock title="Server.js" lang="javascript" copyable>
const port = 3000;
function listen() {
  console.log("listening on", port);
}
</d-codeblock>

<!-- line highlighting and line numbers -->
<d-codeblock lang="javascript" highlight="2,4-5">
const a = 1;
const b = 2;
const c = 3;
const sum = a + b + c;
console.log(sum);
</d-codeblock>

<!-- diff mode with floating copy button -->
<d-codeblock diff copyable>
 initial config
-const timeout = 1000;
+const timeout = 5000;
 return timeout;
</d-codeblock>
```

<a name="attributes-and-properties-d-codeblock"></a>
#### attributes and properties

| attribute      | property      | type      | default | form | description                                                           |
| :------------- | :------------ | :-------- | :------ | :--- | :-------------------------------------------------------------------- |
| `title`        | `title`       | `string`  | `""`    | no   | filename or label displayed in the header bar                         |
| `lang`         | `lang`        | `string`  | `""`    | no   | code language identifier displayed as an uppercase badge              |
| `code`         | `code`        | `string`  | `""`    | no   | programmatic code string input                                        |
| `line-numbers` | `lineNumbers` | `boolean` | `true`  | no   | whether to render line numbers beside each row                        |
| `highlight`    | `highlight`   | `string`  | `""`    | no   | comma separated line numbers and ranges to highlight (e.g. `"2,4-6"`) |
| `diff`         | `diff`        | `boolean` | `false` | no   | enables diff styling for lines starting with `+` or `-`               |
| `copyable`     | `copyable`    | `boolean` | `true`  | no   | whether to display a copy button (in header or floating top right)    |
| `wrap`         | `wrap`        | `boolean` | `false` | no   | wraps long lines instead of horizontal scrolling                      |
| `max-height`   | `maxHeight`   | `string`  | `""`    | no   | maximum scrollable height for long code snippets                      |
| `code-font-size` | `codeFontSize` | `string` | `""`   | no   | custom font size applied specifically to code text                    |

<a name="programmatic-api-d-codeblock"></a>
#### programmatic api

```javascript
const block = select('d-codeblock');

// trigger copy programmatically
block.copy();

// read plain code text
block.codeText; // string
```

<a name="events-d-codeblock"></a>
#### events

| event  | detail             | bubbles | description                                     |
| :----- | :----------------- | :------ | :---------------------------------------------- |
| `copy` | `{ text: string }` | yes     | dispatched when code is copied to the clipboard |

<a name="behavior-d-codeblock"></a>
#### behavior

- when `title` or `lang` is provided, displays an integrated header bar with file icon, filename, language badge, and copy button.
- when neither `title` nor `lang` is provided, renders a copy button anchored in the top right corner.
- animated copy action transitions the copy icon into a green checkmark, reverting after 2 seconds.

---

### d-color-picker

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/color-picker.mp4" type="video/mp4">
  </video>
</p>

color picker featuring 2d saturation and brightness controls, hue slider, and alpha transparency controls.

```html
<d-color-picker value="#3b82f6" name="theme_color" label="accent color"></d-color-picker>
```

#### attributes and properties

| attribute | property | type      | default          | form | description                                             |
| :-------- | :------- | :-------- | :--------------- | :--- | :------------------------------------------------------ |
| `value`   | `value`  | `string`  | `"#d3ac5f"`      | yes  | current hex color value (supports 3, 4, 6, 8 digit hex) |
| `name`    | `name`   | `string`  | `""`             | no   | input name attribute for form submission                |
| `id`      | `id`     | `string`  | `""`             | no   | element identifier                                      |
| `label`   | `label`  | `string`  | `"color picker"` | no   | header label displayed above the picker                 |
| `opened`  | `opened` | `boolean` | `false`          | no   | whether the popup picker panel is initially open        |

#### programmatic api

```javascript
const picker = select('d-color-picker');

// open, close, or toggle popup panel visibility
picker.open();
picker.close();
picker.toggle();

// programmatically set hex color
picker.setHex('#ef4444');

// read color state
console.log(picker.hex); // current hex string
console.log(picker.h, picker.s, picker.b, picker.a); // hsb and alpha values
```

#### events

| event    | detail     | bubbles | description                                              |
| :------- | :--------- | :------ | :------------------------------------------------------- |
| `change` | hex string | yes     | dispatched when color changes via drag, input, or setHex |
| `input`  | hex string | yes     | dispatched continuously during dragging                  |

#### behavior

- interactive 2D color plane, hue bar, and alpha slider with checkered transparency.
- mounts its floating picker panel through an internal `d-popover` top layer, flips automatically near viewport edges, stays aligned on scroll, and closes on outside click or `Escape`.
- supports dragging even outside the component bounds.
- automatically redraws on element resize and renders sharply on high resolution displays.
- clicking the reset icon restores the initial color value.

---

### d-combobox

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

#### attributes and properties

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

#### programmatic api

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

#### events

| event    | detail                | bubbles | description                                    |
| :------- | :-------------------- | :------ | :--------------------------------------------- |
| `change` | selected value string | yes     | dispatched when an option is selected or added |
| `open`   | none                  | yes     | dispatched when the dropdown panel opens       |
| `close`  | none                  | yes     | dispatched when the dropdown panel closes      |

#### behavior

- automatically parses child `<option>` tags declared in HTML.
- automatically calculates viewport space to open above or below without overflowing the screen.
- keeps floating coordinates aligned during scrolling and resizing.
- provides full keyboard navigation (arrow keys, home, end, enter, escape).
- displays an empty state indicator when search queries return no matches.

---

### d-context-menu

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/context-menu.mp4" type="video/mp4">
  </video>
</p>

contextual right click menu with boundary clamping and outside click suppression.

```html
<div class="card p-2">
  right click inside this card
  <d-context-menu>
    <span>copy link</span>
    <span>duplicate item</span>
    <span>delete</span>
  </d-context-menu>
</div>
```

#### attributes and properties

| attribute | property | type      | default | form | description                                                     |
| :-------- | :------- | :-------- | :------ | :--- | :-------------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the context menu is open                                |
| `parent`  | `parent` | `string`  | `""`    | no   | selector for target parent element (defaults to parent element) |

#### programmatic api

```javascript
const menu = select('d-context-menu');

// open context menu programmatically (optionally passing pointer event for cursor positioning)
menu.open();
menu.open(pointerEvent);

// close context menu
menu.close();

// toggle context menu
menu.toggle();

// close all open context menus across the document (optionally excluding one instance)
dContextMenu.closeAll();

// listen to item selection (e.detail is the clicked element)
listen(menu, 'click', (e) => {
  const target = e.target;
  const action = target.closest('[data-action]')?.getAttribute('data-action');
  if (action) handleAction(action);
});
```

#### events

| event   | detail          | bubbles | description                                                |
| :------ | :-------------- | :------ | :--------------------------------------------------------- |
| `click` | clicked element | yes     | emitted through component emitter when an item is selected |
| `open`  | none            | yes     | dispatched when context menu opens                         |
| `close` | none            | yes     | dispatched when context menu closes                        |

#### behavior

- opens on right click within its parent container and suppresses the browser default context menu.
- calculates menu dimensions and automatically bounds coordinates so the menu never overflows parent or screen edges.
- clicking outside closes the menu immediately without triggering click actions or links on underlying elements.
- automatically closes any other open context menus before opening.
- when creating dynamically in javascript, listen to `'click'` directly on the component element; item wrappers are created during component mount in the next animation frame.

---

### d-drawer

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/drawer.mp4" type="video/mp4">
  </video>
</p>

offcanvas navigation drawer with animated entrance transitions and wave styling.

```html
<d-drawer direction="left" id="main-drawer">
  <nav class="flex-column g-1 p-2">
    <a href="/dashboard">dashboard</a>
    <a href="/settings">settings</a>
  </nav>
</d-drawer>
```

#### attributes and properties

| attribute   | property    | type      | default  | form | description                                              |
| :---------- | :---------- | :-------- | :------- | :--- | :------------------------------------------------------- |
| `opened`    | `opened`    | `boolean` | `false`  | no   | whether the drawer is open                               |
| `direction` | `direction` | `string`  | `"left"` | no   | entrance edge (`"left"`, `"right"`, `"top"`, `"bottom"`) |

#### programmatic api

```javascript
const drawer = select('d-drawer');

// open drawer (returns promise that resolves after animation completes)
await drawer.open();

// close drawer (returns promise)
await drawer.close();

// toggle drawer
await drawer.toggle();

// inspect animation and open state
console.log(drawer.isOpened, drawer.isAnimating);
```

#### events

| event   | detail | bubbles | description                                     |
| :------ | :----- | :------ | :---------------------------------------------- |
| `open`  | none   | yes     | dispatched when the opening animation completes |
| `close` | none   | yes     | dispatched when the closing animation completes |

#### behavior

- slides into view from the configured edge with multi layer wave animations.
- programmatic `open()`, `close()`, and `toggle()` methods return promises that resolve upon animation completion.

---

### d-dropdown

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/dropdown.mp4" type="video/mp4">
  </video>
</p>

collapsible accordion dropdown container, perfect for faqs.

```html
<d-dropdown header="account security">
  <div class="p-1 flex-column g-1">
    <d-checkbox text="two factor authentication"></d-checkbox>
    <a href="/sessions" class="btn fs-08">manage active sessions</a>
  </div>
</d-dropdown>
```

#### attributes and properties

| attribute | property | type      | default      | form | description                                      |
| :-------- | :------- | :-------- | :----------- | :--- | :----------------------------------------------- |
| `header`  | `header` | `string`  | `"dropdown"` | no   | title text displayed on the clickable header bar |
| `opened`  | `opened` | `boolean` | `false`      | no   | whether the dropdown is expanded                 |

#### programmatic api

```javascript
const dropdown = select('d-dropdown');

// open dropdown
dropdown.open();

// close dropdown
dropdown.close();

// toggle dropdown
dropdown.toggle();

// inspect open state
console.log(dropdown.isOpened);
```

#### events

| event   | detail | bubbles | description                               |
| :------ | :----- | :------ | :---------------------------------------- |
| `open`  | none   | yes     | dispatched when the dropdown panel opens  |
| `close` | none   | yes     | dispatched when the dropdown panel closes |

#### behavior

- clicking the header expands or collapses child content with an animated chevron indicator.
- dynamically updates header text and open state when properties change.

---

### d-file-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/file-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop file upload container with interactive item list, formatted byte size calculations, and compact bar modes.

```html
<d-file-input accept=".pdf,.zip,.csv" name="documents" multiple compact></d-file-input>
```

#### attributes and properties

| attribute     | property      | type             | default                | form | description                                  |
| :------------ | :------------ | :--------------- | :--------------------- | :--- | :------------------------------------------- |
| `accept`      | `accept`      | `string`         | `"*/*"`                | no   | allowed mime types or file extensions        |
| `icon`        | `icon`        | `string`         | `"dstrn-folder-line"`  | no   | icon class displayed in dropzone             |
| `placeholder` | `placeholder` | `string`         | `"drag & drop file"`   | no   | primary dropzone title                       |
| `subtitle`    | `subtitle`    | `string`         | `"or click to browse"` | no   | secondary helper subtitle                    |
| `compact`     | `compact`     | `boolean`        | `true`                 | no   | renders as a slim horizontal bar when true   |
| `multiple`    | `multiple`    | `boolean`        | `false`                | no   | allows selecting and dropping multiple files |
| `name`        | `name`        | `string`         | `""`                   | no   | form payload field name                      |
| `id`          | `id`          | `string`         | `""`                   | no   | component id                                 |
| `value`       | `value`       | `object`/`array` | `null`                 | yes  | `File`, `File[]`, or url string payload      |

#### programmatic api

```javascript
const fileInput = select('d-file-input');

// reset dropzone, clearing value, file list, and native input
fileInput.reset();

// inspect selected files
console.log(fileInput.value); // File instance or File[] array
```

#### events

| event    | detail                      | bubbles | description                                             |
| :------- | :-------------------------- | :------ | :------------------------------------------------------ |
| `change` | `File`, `File[]`, or `null` | yes     | dispatched when files are selected, dropped, or deleted |

#### behavior

- drag and drop file upload with file list display, formatted sizes (Bytes, KB, MB, GB), and delete buttons.
- automatically clears error states when a valid file is dropped.
- integrates with standard form submission.

---

### d-form

ajax form container with automatic csrf token injection, client side validation, inline field error rendering, and response routing.

```html
<d-form action="/register" method="POST" rules='{"email": "required|email"}' class="flex-column g-1">
  <input type="email" name="email" placeholder="email address">
  <input type="password" name="password" placeholder="password">
  <button type="submit" class="btn">register</button>
</d-form>
```

<a name="attributes-and-properties-d-form"></a>

#### attributes and properties

| attribute      | property      | type      | default  | form | description                                                        |
| :------------- | :------------ | :-------- | :------- | :--- | :----------------------------------------------------------------- |
| `action`       | `action`      | `string`  | `""`     | no   | submission destination url                                         |
| `method`       | `method`      | `string`  | `"POST"` | no   | http request method (`GET`, `POST`, `PUT`, `DELETE`)               |
| `rules`        | `rules`       | `string`  | `""`     | no   | json string of client side validation rules                        |
| `target`       | `target`      | `string`  | `""`     | no   | css selector of element to swap when server returns html           |
| `navigate`     | `navigate`    | `boolean` | `false`  | no   | updates browser history and uses spa router when html is returned  |
| `force-reload` | `forceReload` | `string`  | `""`     | no   | reload condition (`""`, `"true"`, `"all"`, `"success"`, `"error"`) |
| `callback`     | `callback`    | `string`  | `""`     | no   | name of global window function executed after successful response  |

<a name="programmatic-api-d-form"></a>

#### programmatic api

```javascript
const form = select('d-form');

// submit form programmatically
form.submit();

// retrieve form data
const formData = form.serialize();

// display validation errors manually
form.showErrors({ email: ['invalid address'] });

// clear active errors
form.clearErrors();

// disable or enable form controls
form.setDisabled(true);
```

<a name="events-d-form"></a>

#### events

| event        | detail               | bubbles | description                                              |
| :----------- | :------------------- | :------ | :------------------------------------------------------- |
| `d-submit`   | none                 | yes     | cancelable, dispatched before submission starts          |
| `d-success`  | `object` or `string` | yes     | dispatched when request succeeds                         |
| `d-error`    | `object`             | yes     | dispatched when request fails or validation errors occur |
| `d-response` | `object` or `string` | yes     | dispatched on every completed response                   |

<a name="behavior-d-form"></a>

#### behavior

- wraps or injects an internal form element and intercepts the submit event.
- injects `X-Requested-With: dSPAHttpRequest` and `X-CSRF-TOKEN` headers automatically.
- disables all inputs, buttons, and form controls during in flight submission to prevent duplicate requests.
- evaluates client side rules if defined (prevents submission and injects inline field errors if validation fails).
- for file uploads, sends `multipart/form-data` (for standard fields, sends `application/x-www-form-urlencoded`).

<a name="response-lifecycle"></a>

#### response lifecycle

| controller return                  | status / type                             | default behavior (no force-reload)                                                                                                   | behavior with force-reload                                                                    |
| :--------------------------------- | :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| `return json({ data })`            | `200` (`application/json`)                | parses payload, dispatches `d-success` and `d-response`, executes `callback` if defined, leaves form on page without reload          | reloads page if set to `force-reload`, `all`, or `success` (ignored on `error`)               |
| `return json({ redirect })`        | `200` (`application/json`)                | navigates via `dSPA.navigate(redirect)` without reload (executes full page redirect if `force: true` or `X-Redirect-Force` is set)   | navigates via spa router (or hard redirect if `force: true`)                                  |
| `return json().withErrors(errors)` | `422` (`application/json`)                | marks inputs with `.error`, injects `.error-message` below fields (or `.error-banner` at top), dispatches `d-error` and `d-response` | reloads page if set to `force-reload`, `all`, or `error` (retains inline errors on `success`) |
| `validate()` failure               | `422` (`application/json`)                | renders field validation errors inline, dispatches `d-error` and `d-response`, keeps form state intact                               | reloads page if set to `force-reload`, `all`, or `error` (retains inline errors on `success`) |
| `return abort(status, message)`    | `4xx` or `5xx` (`application/json`)       | displays `.error-banner` at top of form, dispatches `d-error` and `d-response`, reenables form inputs                                | reloads page if set to `force-reload`, `all`, or `error` (displays banner on `success`)       |
| `return render(view, data)`        | `200` (`text/html`)                       | swaps returned markup into target selector (default `body`), runs scripts, dispatches `d-success` (updates url if `navigate` is set) | reloads page if set to `force-reload`, `all`, or `success` (proceeds with swap on `error`)    |
| unexpected server error            | `500` (`text/html` or `application/json`) | renders full 500 error page via `document.write` for html (displays error banner for json)                                           | displays 500 error page for html (reloads page for json if set to `all` or `error`)           |
| unexpected 404 error               | `404` (`text/html` or `application/json`) | swaps target with 404 error template for html (displays error banner for json)                                                       | reloads page if set to `all` or `error` (preserves view on `success`)                         |

---

### d-hamburger

responsive mobile navigation hamburger trigger and full screen navigation overlay with programmable breakpoints and multi level sliding subcategory panels.

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <a d-link href="/" class="flex-row align-center">
      <img src="/img/logo.png" alt="logo">
    </a>

    <div id="nav-links" class="flex-row align-center g-2 md:d-flex d-none">
      <a d-link href="/features" class="nav-link">features</a>
      <div d-submenu="framework">
        <a d-link href="/architecture">architecture</a>
        <a d-link href="/benchmarks">benchmarks</a>
      </div>
    </div>

    <d-hamburger target="#nav-links" breakpoint="md">
      <div slot="header">
        <img src="/img/logo.png" class="logo-mark">
      </div>
      <div slot="footer">
        <d-text-input placeholder="search docs"></d-text-input>
      </div>
    </d-hamburger>
  </div>
</nav>
```

#### attributes and properties

| attribute        | property         | type      | default           | form | description                                                              |
| :--------------- | :--------------- | :-------- | :---------------- | :--- | :----------------------------------------------------------------------- |
| `opened`         | `opened`         | `boolean` | `false`           | no   | whether the overlay is open                                              |
| `breakpoint`     | `breakpoint`     | `string`  | `"md"`            | no   | breakpoint token (`sm`, `md`, `lg`, `xl`, `xxl`, `ultra`) or media query |
| `target`         | `target`         | `string`  | `""`              | no   | selector of desktop container to clone links from                        |
| `animation`      | `animation`      | `string`  | `"slide-down"`    | no   | overlay transition (`slide-down`, `fade`, `slide-right`, `slide-left`)   |
| `direction`      | `direction`      | `string`  | `"top"`           | no   | entrance direction                                                       |
| `size`           | `size`           | `string`  | `"1.5em"`         | no   | hamburger button size in em units                                        |
| `color`          | `color`          | `string`  | `"currentColor"`  | no   | button stroke color                                                      |
| `activecolor`    | `activeColor`    | `string`  | `"var(--accent)"` | no   | button color when open                                                   |
| `autohidetarget` | `autoHideTarget` | `boolean` | `true`            | no   | automatically hides desktop target container below breakpoint            |
| `title`          | `title`          | `string`  | `""`              | no   | fallback header title when `slot="header"` is omitted                    |

#### slots

- `slot="header"`: custom header content in the mobile overlay.
- `slot="footer"`: custom footer content at the bottom of the overlay.

#### programmatic api

```javascript
const hamburger = select('d-hamburger');

// open overlay (returns promise)
await hamburger.open();

// close overlay (returns promise)
await hamburger.close();

// toggle overlay
await hamburger.toggle();

// navigate into a specific subcategory panel programmatically
hamburger.navigateTo(subPanelElement, 'framework');

// navigate back to previous parent panel
hamburger.navigateBack();

// close all open hamburger instances across the page
dHamburger.closeAll();
```

#### events

| event   | detail | bubbles | description                    |
| :------ | :----- | :------ | :----------------------------- |
| `open`  | none   | yes     | dispatched when overlay opens  |
| `close` | none   | yes     | dispatched when overlay closes |

#### behavior

- automatically displays the mobile button and hides the desktop navigation when below the specified breakpoint.
- supports nested sliding subpanels using `d-submenu` attributes on link containers.
- automatically manages page scroll locking while open.
- closes automatically when clicking links, pressing Escape, or navigating between pages.

---

### d-hold-button

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

#### attributes and properties

| attribute  | property   | type      | default    | form | description                                                     |
| :--------- | :--------- | :-------- | :--------- | :--- | :-------------------------------------------------------------- |
| `delay`    | `delay`    | `number`  | `800`      | no   | hold duration in milliseconds required before triggering action |
| `type`     | `type`     | `string`  | `"button"` | no   | button type (`"button"`, `"submit"`)                            |
| `disabled` | `disabled` | `boolean` | `false`    | no   | whether the button is disabled                                  |

#### programmatic api

```javascript
const holdBtn = select('d-hold-button');

// adjust required hold duration in ms
holdBtn.delay = 2000;

// toggle disabled state
holdBtn.disabled = true;
```

#### events

| event   | detail             | bubbles | description                                               |
| :------ | :----------------- | :------ | :-------------------------------------------------------- |
| `click` | native click event | yes     | dispatched only after holding for the full delay duration |

#### behavior

- requires continuous press for the configured duration before triggering the click action.
- releasing before completion cancels the action and resets visual progress.
- automatically submits the surrounding form when configured with `type="submit"`.

---

### d-icon-button

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/icon-button.mp4" type="video/mp4">
  </video>
</p>

icon button wrapper supporting custom icons, sizes, and dynamic theme active colors.

```html
<d-icon-button icon="dstrn-heart-line" size="2em" color="var(--content-l)" activecolor="var(--red)"></d-icon-button>
```

#### attributes and properties

| attribute     | property      | type     | default              | form | description                                              |
| :------------ | :------------ | :------- | :------------------- | :--- | :------------------------------------------------------- |
| `icon`        | `icon`        | `string` | `""`                 | no   | css icon class (e.g. `dstrn-heart-line`, `dstrn-search`) |
| `size`        | `size`        | `string` | `"4.2em"`            | no   | font size of the icon container                          |
| `color`       | `color`       | `string` | `"var(--content-l)"` | no   | default icon color                                       |
| `activecolor` | `activeColor` | `string` | `"var(--accent)"`    | no   | active icon color                                        |
| `id`          | `id`          | `string` | `""`                 | no   | component id                                             |

---

### d-image-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/image-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop image dropzone with instant client preview and action overlay.

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="cover" placeholder="drop profile photo"></d-image-input>
```

#### attributes and properties

| attribute      | property      | type              | default                              | form | description                                          |
| :------------- | :------------ | :---------------- | :----------------------------------- | :--- | :--------------------------------------------------- |
| `accept`       | `accept`      | `string`          | `"image/png, image/jpeg, image/gif"` | no   | allowed image mime types                             |
| `icon`         | `icon`        | `string`          | `"dstrn-pic-line"`                   | no   | placeholder icon class                               |
| `placeholder`  | `placeholder` | `string`          | `"drag & drop image"`                | no   | dropzone title text                                  |
| `subtitle`     | `subtitle`    | `string`          | `"or click to browse"`               | no   | dropzone subtitle text                               |
| `replace-text` | `replaceText` | `string`          | `"replace"`                          | no   | text for overlay replace button                      |
| `delete-text`  | `deleteText`  | `string`          | `"delete"`                           | no   | text for overlay delete button                       |
| `no-replace`   | `noReplace`   | `boolean`         | `false`                              | no   | hides the replace button when true                   |
| `no-delete`    | `noDelete`    | `boolean`         | `false`                              | no   | hides the delete button when true                    |
| `fit`          | `fit`         | `string`          | `"contain"`                          | no   | object-fit CSS property (`contain`, `cover`, `fill`) |
| `name`         | `name`        | `string`          | `""`                                 | no   | form payload field name                              |
| `id`           | `id`          | `string`          | `""`                                 | no   | component id                                         |
| `value`        | `value`       | `object`/`string` | `null`                               | yes  | `File` object or image URL string                    |

#### programmatic api

```javascript
const imgInput = select('d-image-input');

// reset dropzone and clear image preview
imgInput.reset();

// get or set preview src directly
imgInput.preview = '/img/avatar.png';

// read selected File instance
console.log(imgInput.value);
```

#### events

| event    | detail           | bubbles | description                                               |
| :------- | :--------------- | :------ | :-------------------------------------------------------- |
| `change` | `File` or `null` | yes     | dispatched when an image is selected, dropped, or deleted |

#### behavior

- provides drag and drop image selection with instant preview and action buttons to replace or remove images.
- automatically integrates with form submission and clears error states on upload.

---

### d-key

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
#### attributes and properties

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
#### programmatic api

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
#### events

| event    | detail                    | bubbles | description                                                         |
| :------- | :------------------------ | :------ | :------------------------------------------------------------------ |
| `change` | string (normalized chord) | yes     | dispatched when a new shortcut is recorded or cleared in input mode |

<a name="behavior-d-key"></a>
#### behavior

- parses `+` or `,` separated chords from either the `value` attribute or initial text content and always exposes the normalized string on `.value` and token list on `.keys`.
- renders platform aware glyphs (`cmd` and `mod` display `⌘` on mac and `ctrl` elsewhere, `shift` displays `⇧`, `alt` displays `⌥` on mac, `enter` displays `⏎`).
- in `editable` mode, clicking or focusing starts recording; modifiers preview live, completing with a key commits and emits `change`, `Escape` cancels, and `Backspace`/`Delete` clears when `clearable` is enabled.
- integrates inside `.btn` elements and groups with `.d-key-group`.

---

### d-loader

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/loader.mp4" type="video/mp4">
  </video>
</p>

canvas arc spinner with automatic viewport intersection pause.

```html
<d-loader style="width: 2em; height: 2em;"></d-loader>
```

#### attributes and properties

| attribute | property | type     | default | form | description                                                  |
| :-------- | :------- | :------- | :------ | :--- | :----------------------------------------------------------- |
| `color`   | `color`  | `string` | `""`    | no   | stroke color (defaults to `--accent` or computed text color) |

#### programmatic api

```javascript
const loader = select('d-loader');

// destroy loader and disconnect observers
loader.destroy();
```

#### behavior

- lightweight canvas spinner that automatically pauses animation when scrolled offscreen to conserve CPU.
- automatically adapts to container dimensions and renders sharply on high resolution screens.

---

### d-link

client side navigation link component and directive that intercepts clicks, fetches content asynchronously, and performs dom swaps without full page reloads.

```html
<!-- custom element -->
<d-link href="/dashboard" class="nav-link">dashboard</d-link>

<!-- anchor directive alias -->
<a d-link href="/settings" target="#settings-panel" mode="inner">settings</a>
```

<a name="attributes-and-properties-d-link"></a>

#### attributes and properties

| attribute         | property         | type      | default   | form | description                                                            |
| :---------------- | :--------------- | :-------- | :-------- | :--- | :--------------------------------------------------------------------- |
| `href`            | `href`           | `string`  | `""`      | no   | target destination url                                                 |
| `target`          | `target`         | `string`  | `""`      | no   | comma separated css selectors of elements to swap in current page      |
| `src`             | `src`            | `string`  | `""`      | no   | comma separated css selectors of elements to extract from fetched page |
| `mode`            | `mode`           | `string`  | `"inner"` | no   | content swap mode (`"inner"`, `"outer"`, `"append"`, `"prepend"`)      |
| `preserve-scroll` | `preserveScroll` | `boolean` | `false`   | no   | preserves window scroll position across navigation                     |
| `d-full-reload`   | `dFullReload`    | `boolean` | `false`   | no   | bypasses client side router and triggers standard browser reload       |

<a name="programmatic-api-d-link"></a>

#### programmatic api

```javascript
// navigate programmatically using dSPA
await dSPA.navigate('/dashboard', {
  targets: ['#content'],
  mode: 'inner',
  preserveScroll: true
});
```

<a name="behavior-d-link"></a>

#### behavior

- intercepts click events on `<d-link>` elements and `<a d-link>` anchors.
- ignores clicks with modifier keys (cmd, ctrl, shift, alt), target `_blank`, hash links, or external protocol urls (mailto, tel, javascript).
- validates same origin before initiating fetch requests.
- falls back to full browser navigation if network fetch fails.

---

### d-menubar

desktop style menu bar with top level triggers, instant hover switching in active mode, nested submenus, checkable items, radio groups, and integrated keyboard shortcuts.

```html
<d-menubar>
  <div d-menu="file">
    <span slot="trigger">file</span>
    <div slot="content">
      <div d-item value="new-tab" shortcut="cmd+t">new tab</div>
      <div d-item value="new-window" shortcut="cmd+n">new window</div>
      <div d-item value="incognito" disabled>new incognito window</div>
      <hr>
      <div d-submenu="share">
        <span slot="trigger">share</span>
        <div slot="content">
          <div d-item value="email-link">email link</div>
          <div d-item value="copy-link" shortcut="cmd+c">copy link</div>
        </div>
      </div>
      <hr>
      <div d-item value="print" shortcut="cmd+p">print</div>
    </div>
  </div>
  <div d-menu="edit">
    <span slot="trigger">edit</span>
    <div slot="content">
      <div d-item value="undo" shortcut="cmd+z">undo</div>
      <div d-item value="redo" shortcut="cmd+shift+z">redo</div>
    </div>
  </div>
  <div d-menu="view">
    <span slot="trigger">view</span>
    <div slot="content">
      <div d-check value="show-bookmarks" checked shortcut="cmd+b">show bookmarks</div>
      <div d-check value="show-full-urls">show full urls</div>
    </div>
  </div>
  <div d-menu="profiles">
    <span slot="trigger">profiles</span>
    <div slot="content">
      <div d-radio-group name="profile" value="work">
        <div d-radio value="personal">personal</div>
        <div d-radio value="work">work</div>
        <div d-radio value="guest">guest</div>
      </div>
    </div>
  </div>
</d-menubar>
```

<a name="attributes-and-properties-d-menubar"></a>
#### attributes and properties

| attribute  | property   | type      | default | form | description                                                                |
| :--------- | :--------- | :-------- | :------ | :--- | :------------------------------------------------------------------------- |
| `items`    | `items`    | `array`   | `null`  | no   | structured menu definition array (alternative to declarative child markup) |
| `disabled` | `disabled` | `boolean` | `false` | no   | disables menubar interaction                                               |

declarative markup attributes:

- `d-menu="<id>"`: top level menu container (child of `<d-menubar>`)
- `slot="trigger"`: trigger element label or button inside `[d-menu]` or `[d-submenu]`
- `slot="content"`: dropdown panel wrapper containing items
- `d-item`: action item (`value` attribute defines identifier, `shortcut` renders `<d-key>` badge, `disabled` disables)
- `d-check`: toggleable check item (`value`, `checked`, `shortcut`, `disabled`)
- `d-radio-group`: mutual exclusion group (`name` and `value` attributes) containing `[d-radio]` items
- `d-submenu="<id>"`: nested submenu container containing `slot="trigger"` and `slot="content"`
- `<hr>` (or `.d-menubar-sep`): visual hairline separator divider

<a name="programmatic-api-d-menubar"></a>
#### programmatic api

```javascript
const bar = select('d-menubar');

bar.openMenu(0);
bar.closeAllMenus();
bar.items = [
  {
    id: 'file',
    title: 'file',
    items: [
      { value: 'new', label: 'new tab', shortcut: 'cmd+t' },
      'separator',
      { value: 'save', label: 'save', shortcut: 'cmd+s' }
    ]
  }
];

// close all menubar instances
dMenubar.closeAll();
```

<a name="events-d-menubar"></a>
#### events

| event    | detail                                    | bubbles | description                                             |
| :------- | :---------------------------------------- | :------ | :------------------------------------------------------ |
| `select` | `{ value, item, menu }`                   | yes     | dispatched when an action item is invoked               |
| `change` | `{ value, checked }` or `{ name, value }` | yes     | dispatched when a check item or radio selection changes |
| `open`   | `{ menu, index }`                         | yes     | dispatched when a top level menu panel opens            |
| `close`  | `{ menu, index }`                         | yes     | dispatched when a menu panel closes                     |

<a name="behavior-d-menubar"></a>
#### behavior

- renders dropdown panels in the browser top layer via `d-popover`, avoiding clipping inside nested overflow containers.
- clicking any top level menu opens the panel and activates menu mode; moving the cursor across sibling top triggers switches between menus.
- submenus reveal on pointer hover with grace delay and support nested hierarchies.
- supports complete keyboard navigation (`ArrowLeft` / `ArrowRight` between top triggers, `ArrowDown` / `ArrowUp` through menu items, `ArrowRight` / `ArrowLeft` into and out of submenus, `Enter` / `Space` to invoke, `Escape` to close).

---

### d-modal

modal dialog container with page scroll locking and backdrop dismiss protection.

```html
<d-modal id="confirm-modal" persistent>
  <div class="flex-column g-1">
    <h3>confirm deletion</h3>
    <p class="text-content-l">this action cannot be undone.</p>
    <div class="flex-row justify-end g-1 mt-1">
      <button class="btn fs-08" onclick="this.closest('d-modal').close()">cancel</button>
      <button class="btn btn-red fs-08">delete</button>
    </div>
  </div>
</d-modal>
```

#### attributes and properties

| attribute    | property     | type      | default | form | description                                                           |
| :----------- | :----------- | :-------- | :------ | :--- | :-------------------------------------------------------------------- |
| `opened`     | `opened`     | `boolean` | `false` | no   | whether the modal is currently visible                                |
| `persistent` | `persistent` | `boolean` | `false` | no   | whether the element remains in the dom when closed via backdrop click |

#### programmatic api

```javascript
const modal = select('d-modal');

// open modal (locks body scroll)
modal.open();

// close modal (runs exit animation and unlocks body scroll)
modal.close();

// close, await exit transition, and remove element from dom
await modal.destroy();
```

#### events

| event   | detail | bubbles | description                  |
| :------ | :----- | :------ | :--------------------------- |
| `open`  | none   | yes     | dispatched when modal opens  |
| `close` | none   | yes     | dispatched when modal closes |

#### behavior and styling boundaries

- displays an overlay dialog and prevents page scrolling while visible.
- when `persistent` is omitted (or `false`), clicking the backdrop closes and destroys the element (`destroy()`), removing it from the dom.
- when `persistent` is present (`<d-modal persistent>`), clicking the backdrop closes the dialog (`close()`) without removing the element from the dom, allowing it to be reopened.
- `<d-modal>` automatically injects `.d-modal-wrapper` (which provides `var(--container-l)` background, border, radius and shadow) and `.d-modal-content` (flex container with scroll management).
- to customize or restyle the modal appearance, target `d-modal .d-modal-wrapper` or `d-modal .d-modal-content` in your css.

---

### d-morph

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/morph.mp4" type="video/mp4">
  </video>
</p>

physics based morphing animation engine (`dMotion`) that transitions between named elements anywhere in the document with continuous geometry, border-radius, box-shadow, and form state interpolation.

```html
<d-morph name="summary-card" d-action="click" d-to="detail-card" d-duration="350" d-visible>
  <div class="card p-2">
    <h4>project summary</h4>
    <p>click to expand details</p>
  </div>
</d-morph>

<d-morph name="detail-card" d-action="click" d-to="summary-card" d-duration="350">
  <div class="card p-4">
    <h4>detailed project view</h4>
    <p>full metadata and extended controls</p>
  </div>
</d-morph>
```

#### attributes and properties

| attribute    | property | type      | default  | description                                                   |
| :----------- | :------- | :-------- | :------- | :------------------------------------------------------------ |
| `name`       | `name`   | `string`  | `""`     | unique morph identifier registered in `window.dMotion`        |
| `d-visible`  | none     | `boolean` | `false`  | marks this morph element as initially active and visible      |
| `d-action`   | none     | `string`  | `""`     | trigger actions: `"click"`, `"long-press"`, `"click-out"`     |
| `d-to`       | none     | `string`  | `""`     | target morph name to transition to                            |
| `d-duration` | none     | `number`  | computed | transition duration in ms (computed from distance if omitted) |

#### programmatic api (`window.dMotion`)

```javascript
// get a morph element instance by name
const morph = dMotion.get('summary-card');

// get the immediately previous morph element
const prev = dMotion.getPrevious();

// programmatically trigger a transition between two named elements
await dMotion.transition('summary-card', 'detail-card', optionalTriggerNode);

// register transition lifecycle hooks ('before' or 'after')
dMotion.listen('summary-card', 'detail-card', () => {
  console.log('transition starting');
}, 'before');

// unregister hook
dMotion.unlisten('summary-card', 'detail-card', callback, 'before');

// component methods on <d-morph>
morph.show();
morph.hide();
```

#### behavior

- creates smooth animated morph transitions between two elements anywhere on the page.
- preserves visual styles (size, position, border radius, shadows) and element states during animation.
- supports click, long-press, and click-out triggers.
- child elements with `d-no-morph` remain interactive without triggering the transition.

---

### d-navmenu

horizontal website navigation menu featuring a tooltips, shared morphing viewport panel with directional cross slide transitions, and drilldown integration with `d-hamburger` on mobile.

```html
<nav class="nav">
  <div class="container flex-row align-center justify-between py-1">
    <d-navmenu id="site-nav">
      <a d-link href="/" class="nav-link">overview</a>
      <div d-submenu="getting started">
        <span slot="trigger">getting started</span>
        <div slot="content" class="flex-column g-05" style="width: 280px;">
          <a d-link href="/docs" class="d-navmenu-sublink">
            <span class="d-navmenu-sublink-title">introduction</span>
            <span class="d-navmenu-sublink-desc">reusable frontend component library</span>
          </a>
          <a d-link href="/installation" class="d-navmenu-sublink">
            <span class="d-navmenu-sublink-title">installation</span>
            <span class="d-navmenu-sublink-desc">quick start and directory structure</span>
          </a>
        </div>
      </div>
      <div d-submenu="components">
        <span slot="trigger">components</span>
        <div slot="content" class="flex-row g-1" style="width: 380px;">
          <div class="flex-column g-05 w-50">
            <a d-link href="/popover" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">popover</span>
              <span class="d-navmenu-sublink-desc">top layer floating overlay</span>
            </a>
            <a d-link href="/tooltip" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">tooltip</span>
              <span class="d-navmenu-sublink-desc">hover hint with shortcut badge</span>
            </a>
          </div>
          <div class="flex-column g-05 w-50">
            <a d-link href="/menubar" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">menubar</span>
              <span class="d-navmenu-sublink-desc">desktop application bar</span>
            </a>
            <a d-link href="/otp" class="d-navmenu-sublink">
              <span class="d-navmenu-sublink-title">otp</span>
              <span class="d-navmenu-sublink-desc">one time verification code</span>
            </a>
          </div>
        </div>
      </div>
      <a d-link href="/docs" class="nav-link">documentation</a>
    </d-navmenu>

    <!-- mobile hamburger pairing: automatically converts d-navmenu into drilldown panels -->
    <d-hamburger target="#site-nav" breakpoint="md">
      <div slot="header">
        <span class="fs-09 text-content-l">menu</span>
      </div>
    </d-hamburger>
  </div>
</nav>
```

<a name="attributes-and-properties-d-navmenu"></a>
#### attributes and properties

| attribute     | property      | type      | default        | form | description                                                                    |
| :------------ | :------------ | :-------- | :------------- | :--- | :----------------------------------------------------------------------------- |
| `items`       | `items`       | `array`   | `null`         | no   | structured items array (alternative to declarative children)                   |
| `delay`       | `delay`       | `number`  | `150`          | no   | hover grace delay in milliseconds before closing                               |
| `orientation` | `orientation` | `string`  | `"horizontal"` | no   | menu layout orientation (`"horizontal"`, `"vertical"`)                         |
| `align`       | `align`       | `string`  | `"center"`     | no   | viewport alignment relative to active trigger (`"center"`, `"start"`, `"end"`) |
| `disabled`    | `disabled`    | `boolean` | `false`        | no   | disables navigation interaction                                                |

declarative markup attributes:

- `d-submenu="<title>"`: dropdown panel container (child of `<d-navmenu>`)
- `slot="trigger"`: trigger label or markup inside `[d-submenu]`
- `slot="content"`: dropdown panel contents wrapper
- `.d-navmenu-sublink`: helper card link with `.d-navmenu-sublink-title` and `.d-navmenu-sublink-desc`

<a name="programmatic-api-d-navmenu"></a>
#### programmatic api

```javascript
const nav = select('d-navmenu');

nav.open(1);
nav.close();
nav.isOpened;
nav.activeIndex;

// retrieve original element clones for mobile menus
const items = nav.getHamburgerItems();

// close all navmenu instances
dNavMenu.closeAll();
```

<a name="events-d-navmenu"></a>
#### events

| event   | detail                   | bubbles | description                           |
| :------ | :----------------------- | :------ | :------------------------------------ |
| `open`  | `{ index, title, item }` | yes     | dispatched when a submenu opens       |
| `close` | none                     | yes     | dispatched when active submenu closes |

<a name="behavior-and-d-hamburger-integration"></a>
#### behavior and d-hamburger integration

- shared morphing viewport panel animates width and height while cross sliding contents between navigation tabs.
- when `<d-hamburger target="#nav-id" breakpoint="md">` targets `<d-navmenu id="nav-id">`, the desktop navigation bar automatically hides on mobile viewports and converts all direct links and `[d-submenu]` sections into drilldown mobile drawer panels with no duplicate markup.
- full keyboard navigation (`ArrowLeft` / `ArrowRight` between triggers, `ArrowDown` / `Enter` to focus into active panel, `Escape` to close).

---

### d-notification

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/notification.mp4" type="video/mp4">
  </video>
</p>

toast notification component with linear progress bar countdown and automated exit lifecycle.

```html
<d-notification timer="4000" opened>
  <p>project settings saved successfully</p>
</d-notification>
```

#### attributes and properties

| attribute | property | type      | default | form | description                                                  |
| :-------- | :------- | :-------- | :------ | :--- | :----------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the toast notification is displayed                  |
| `timer`   | `timer`  | `number`  | `0`     | no   | auto dismissal duration in milliseconds (`0` disables timer) |

#### programmatic api

```javascript
const toast = select('d-notification');

// show toast and start progress bar countdown
toast.show();

// hide toast
toast.hide();

// hide, wait for animation to finish, and remove from dom
await toast.destroy();
```

#### events

| event   | detail | bubbles | description                         |
| :------ | :----- | :------ | :---------------------------------- |
| `open`  | none   | yes     | dispatched when notification opens  |
| `close` | none   | yes     | dispatched when notification closes |

#### behavior

- displays toast notifications with an optional countdown progress bar that automatically dismisses after the timer expires.

---

### d-otp

form associated one time password and verification code input handled by a single native input for  SMS autofill, paste sanitization, keyboard editing, and segmented slot groups.

```html
<d-otp length="6" separator="3" name="code"></d-otp>
<d-otp length="4" mask name="pin"></d-otp>
<d-otp length="6" pattern="alphanumeric" uppercase separator="2"></d-otp>
```

<a name="attributes-and-properties-d-otp"></a>
#### attributes and properties

| attribute   | property    | type      | default    | form | description                                                                        |
| :---------- | :---------- | :-------- | :--------- | :--- | :--------------------------------------------------------------------------------- |
| `length`    | `length`    | `number`  | `6`        | no   | total number of character slots (1 to 16)                                          |
| `value`     | `value`     | `string`  | `""`       | yes  | sanitized OTP string reflected in form submission                                  |
| `name`      | `name`      | `string`  | `""`       | no   | form field name                                                                    |
| `pattern`   | `pattern`   | `string`  | `"digits"` | no   | allowed character preset (`"digits"`, `"alphanumeric"`, `"alpha"`) or regex string |
| `separator` | `separator` | `string`  | `""`       | no   | group split interval (e.g. `"3"`) or comma separated split indices (e.g. `"2,4"`)  |
| `mask`      | `mask`      | `boolean` | `false`    | no   | masks entered characters as dots for PIN entry                                     |
| `disabled`  | `disabled`  | `boolean` | `false`    | no   | disables editing and excludes the field from form submission                       |
| `invalid`   | `invalid`   | `boolean` | `false`    | no   | applies error border styling to all slots                                          |
| `autofocus` | `autofocus` | `boolean` | `false`    | no   | focuses the OTP input automatically on mount                                       |
| `uppercase` | `uppercase` | `boolean` | `false`    | no   | transforms entered letters to uppercase before validation                          |

<a name="programmatic-api-d-otp"></a>
#### programmatic api

```javascript
const otp = select('d-otp');

otp.focus();
otp.error('invalid verification code');
otp.clearError();
otp.clear();
```

<a name="events-d-otp"></a>
#### events

| event      | detail           | bubbles | description                                         |
| :--------- | :--------------- | :------ | :-------------------------------------------------- |
| `input`    | string (`value`) | yes     | dispatched whenever the sanitized OTP value changes |
| `change`   | string (`value`) | yes     | dispatched when the OTP value updates               |
| `complete` | string (`value`) | yes     | dispatched once whenever all slots are filled       |

<a name="behavior-d-otp"></a>
#### behavior

- uses a single underlying `<input autocomplete="one-time-code">` so mobile autofill, clipboard paste (automatically stripping separators or non matching characters), and backspace work normally.

---

### d-popover

top layer floating popover container. supports click, hover, and manual triggers, automatic viewport flipping, scroll tracking across nested containers, and nested popovers.

```html
<d-popover side="bottom" align="start">
  <button slot="trigger" class="btn btn-outline">open popover</button>
  <div slot="content">
    <div class="d-popover-header">
      <h4 class="d-popover-title">dimensions</h4>
      <p class="d-popover-desc">set the dimensions for the layer.</p>
    </div>
    <d-text-input placeholder="width" value="100%"></d-text-input>
  </div>
</d-popover>

<d-popover trigger="hover" side="top" align="center">
  <button slot="trigger" class="btn btn-gray">hover reveal</button>
  <div slot="content">popover content</div>
</d-popover>
```

<a name="attributes-and-properties-d-popover"></a>
#### attributes and properties

| attribute                | property              | type      | default    | form | description                                                                                      |
| :----------------------- | :-------------------- | :-------- | :--------- | :--- | :----------------------------------------------------------------------------------------------- |
| `opened`                 | `opened`              | `boolean` | `false`    | no   | open state reflected on the host                                                                 |
| `trigger`                | `trigger`             | `string`  | `"click"`  | no   | activation mode (`"click"`, `"hover"`, or `"manual"`)                                            |
| `side`                   | `side`                | `string`  | `"bottom"` | no   | preferred placement (`"bottom"`, `"top"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`) |
| `align`                  | `align`               | `string`  | `"start"`  | no   | alignment along the anchor axis (`"start"`, `"center"`, or `"end"`)                              |
| `offset`                 | `offset`              | `number`  | `8`        | no   | pixel gap between anchor and popover surface                                                     |
| `match-width`            | `matchWidth`          | `boolean` | `false`    | no   | locks popover width to match its anchor element                                                  |
| `hover-delay`            | `hoverDelay`          | `number`  | `120`      | no   | hover open and close grace delay in milliseconds                                                 |
| `target`                 | `target`              | `string`  | `""`       | no   | optional CSS selector for an external anchor element                                             |
| `close-on-click-outside` | `closeOnClickOutside` | `boolean` | `true`     | no   | closes when clicking outside the anchor and popover tree                                         |
| `close-on-escape`        | `closeOnEscape`       | `boolean` | `true`     | no   | closes when pressing `Escape`                                                                    |
| `exclusive`              | `exclusive`           | `boolean` | `true`     | no   | closes unrelated open popovers on open while keeping ancestor popovers open                      |
| `unstyled`               | `unstyled`            | `boolean` | `false`    | no   | removes default surface padding, border, and background for custom overlays                      |

<a name="programmatic-api-d-popover"></a>
#### programmatic api

```javascript
const pop = select('d-popover');

pop.open();
pop.close();
pop.toggle();
pop.reposition();
pop.setAnchor(select('#custom-anchor'));
const anchor = pop.getAnchor();

// close all open popovers
dPopover.closeAll();
```

<a name="events-d-popover"></a>
#### events

| event   | detail | bubbles | description                        |
| :------ | :----- | :------ | :--------------------------------- |
| `open`  | none   | yes     | dispatched when the popover opens  |
| `close` | none   | yes     | dispatched when the popover closes |

<a name="behavior-d-popover"></a>
#### behavior

- renders inside the browser top layer so panels are never clipped by `overflow: hidden` or stacking contexts, and stays locked to its anchor during nested container or page scrolling.
- automatically flips to the opposite side when space is constrained and animates from the resolved anchor edge with a directional spring curve.
- accepts either explicit `slot="trigger"` / `slot="content"` children, positional children (first child as trigger, remaining as content), or an external `target` anchor selector.

---

### d-progress

progress bar indicator with optional header label, percentage readout, custom range bounds, and indeterminate loading shimmer.

```html
<d-progress value="33"></d-progress>
<d-progress value="56" label="upload progress" show-value></d-progress>
<d-progress indeterminate label="syncing assets"></d-progress>
```

<a name="attributes-and-properties-d-progress"></a>
#### attributes and properties

| attribute       | property        | type      | default | form | description                                                   |
| :-------------- | :-------------- | :-------- | :------ | :--- | :------------------------------------------------------------ |
| `value`         | `value`         | `number`  | `0`     | no   | current progress value within `min` and `max`                 |
| `min`           | `min`           | `number`  | `0`     | no   | lower bound for ratio calculation                             |
| `max`           | `max`           | `number`  | `100`   | no   | upper bound representing 100 percent completion               |
| `indeterminate` | `indeterminate` | `boolean` | `false` | no   | switches the bar into continuous indeterminate animation mode |
| `label`         | `label`         | `string`  | `""`    | no   | optional label displayed above the track                      |
| `show-value`    | `showValue`     | `boolean` | `false` | no   | displays the rounded percentage readout above the track       |
| `color`         | `color`         | `string`  | `""`    | no   | custom fill color css value (defaults to `var(--accent)`)     |
| `size`          | `size`          | `string`  | `""`    | no   | custom track height css value (defaults to `0.25em`)          |

<a name="programmatic-api-d-progress"></a>
#### programmatic api

```javascript
const progress = select('d-progress');

progress.value = 72;
console.log(progress.percentage); // 72
console.log(progress.ratio);      // 0.72
```

<a name="events-d-progress"></a>
#### events

| event      | detail           | bubbles | description                                                    |
| :--------- | :--------------- | :------ | :------------------------------------------------------------- |
| `complete` | number (`value`) | yes     | dispatched once when `value` reaches `max` in determinate mode |

<a name="behavior-d-progress"></a>
#### behavior

- automatically pauses indeterminate animations when scrolled offscreen and respects `prefers-reduced-motion`.
- triggers a single subtle sheen pass when transitioning into the completed state.

---

### d-radiobutton

form associated radio button with sibling group coordination, optional description text, placement control, and keyboard arrow navigation.

```html
<d-radiobutton name="plan" value="free" text="starter" description="for personal projects" checked></d-radiobutton>
<d-radiobutton name="plan" value="pro" text="pro" description="unlimited workspaces"></d-radiobutton>
<d-radiobutton name="plan" value="team" text="team" placement="after"></d-radiobutton>
```

<a name="attributes-and-properties-d-radiobutton"></a>
#### attributes and properties

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
#### programmatic api

```javascript
const proRadio = select('d-radiobutton[value="pro"]');
proRadio.checked = true;
proRadio.description = 'priority support included';
```

<a name="events-d-radiobutton"></a>
#### events

| event    | detail           | bubbles | description                                        |
| :------- | :--------------- | :------ | :------------------------------------------------- |
| `change` | string (`value`) | yes     | dispatched when the user selects this radio button |

<a name="behavior-d-radiobutton"></a>
#### behavior

- automatically unchecks sibling `<d-radiobutton>` elements with the same `name` within the same `<form>`, `<d-form>`, or document scope.
- supports keyboard navigation with `ArrowUp`/`ArrowLeft`, `ArrowDown`/`ArrowRight`, `Space`, and `Enter`, skipping disabled options.
- respects `prefers-reduced-motion`.

---

### d-questionnaire

interactive multi step questionnaire composing progress tracking, radio buttons, checkboxes, text inputs, and keyboard shortcuts.

```html
<d-questionnaire name="survey">
  <div d-step name="role" title="what is your role?" type="single" required>
    <div d-option value="dev" text="developer" description="building web backends and software"></div>
    <div d-option value="design" text="designer" description="crafting user interfaces"></div>
  </div>
  <div d-step name="tools" title="which tools do you use?" type="multiple" skippable>
    <div d-option value="git" text="git"></div>
    <div d-option value="docker" text="docker"></div>
  </div>
  <div d-step name="notes" title="any extra thoughts?" type="text" placeholder="write your thoughts"></div>
</d-questionnaire>
```

<a name="attributes-and-properties-d-questionnaire"></a>
#### attributes and properties

| attribute        | property        | type      | default | form | description                                                   |
| :--------------- | :-------------- | :-------- | :------ | :--- | :------------------------------------------------------------ |
| `name`           | `name`          | `string`  | `""`    | no   | form field name                                               |
| `value`          | `value`         | `string`  | `"{}"`  | yes  | serialized json payload of answers (form associated)          |
| `current-step`   | `currentStep`   | `number`  | `0`     | no   | current active step index                                     |
| `auto-advance`   | `autoAdvance`   | `boolean` | `false` | no   | automatically advance on single choice selection              |
| `show-progress`  | `showProgress`  | `boolean` | `true`  | no   | displays the animated progress bar                            |
| `show-shortcuts` | `showShortcuts` | `boolean` | `true`  | no   | displays numeric key badges on options                        |
| `questions`      | `questions`     | `string`  | `""`    | no   | json string defining steps as an alternative to html children |

<a name="programmatic-api-d-questionnaire"></a>
#### programmatic api

```javascript
const qn = select('d-questionnaire');

// navigation methods
qn.next();             // validates and proceeds to next step or submits
qn.prev();             // navigates to previous step
qn.skip();             // skips current step if optional or skippable
qn.goTo(1);            // jumps to specified step index
qn.reset();            // clears answers and resets to step 0

// answers access
const ans = qn.answers;              // returns copy of collected answers
qn.setAnswer('role', 'dev');        // sets answer programmatically
```

<a name="events-d-questionnaire"></a>
#### events

| event    | detail                                     | bubbles | description                                  |
| :------- | :----------------------------------------- | :------ | :------------------------------------------- |
| `step`   | object (`{ step, index, total, answers }`) | yes     | emitted when transitioning to a new step     |
| `answer` | object (`{ name, value, answers }`)        | yes     | emitted whenever an answer is set or toggled |
| `submit` | object (`{ answers, json }`)               | yes     | emitted when completing the final step       |

<a name="behavior-d-questionnaire"></a>
#### behavior

- supports keyboard navigation (numbers `1` through `9` select matching options, `Enter` continues, `Backspace` or `ArrowLeft` goes back).
- validates required steps and triggers a shake animation when attempting to continue without answering.
- serializes collected answers to json and integrates with standard `<form>` and `<d-form>` submissions.

---

### d-sidebar

application shell sidebar designed with offcanvas sliding, icon collapse mode, and responsive mobile drawers.

```html
<d-sidebar id="app-sidebar" collapsible="icon" variant="sidebar" persist="app_sidebar">
  <div slot="header">
    <span class="d-sidebar-label">workspace</span>
  </div>

  <div d-sidebar-group>
    <div d-sidebar-group-label>navigation</div>
    <div d-sidebar-menu>
      <a d-sidebar-item href="/dashboard" active>
        <i class="dstrn-dashboard-line"></i>
        <span class="d-sidebar-label">dashboard</span>
      </a>
      <div d-sidebar-item d-collapsible>
        <div d-sidebar-item-trigger>
          <i class="dstrn-folder-line"></i>
          <span class="d-sidebar-label">projects</span>
          <i class="dstrn-down-small-line d-sidebar-chevron"></i>
        </div>
        <div d-sidebar-sub>
          <a d-sidebar-sub-item href="/projects/active">active</a>
          <a d-sidebar-sub-item href="/projects/archive">archive</a>
        </div>
      </div>
    </div>
  </div>

  <div slot="footer">
    <span class="d-sidebar-label">profile</span>
  </div>
  <div d-sidebar-rail></div>
</d-sidebar>
```

<a name="attributes-and-properties-d-sidebar"></a>
#### attributes and properties

| attribute     | property      | type      | default       | form | description                                                                                       |
| :------------ | :------------ | :-------- | :------------ | :--- | :------------------------------------------------------------------------------------------------ |
| `side`        | `side`        | `string`  | `"left"`      | no   | sidebar position (`"left"` or `"right"`)                                                          |
| `collapsible` | `collapsible` | `string`  | `"offcanvas"` | no   | collapse mode (`"offcanvas"`, `"icon"`, or `"none"`)                                              |
| `variant`     | `variant`     | `string`  | `"sidebar"`   | no   | appearance variant (`"sidebar"`, `"floating"`, or `"inset"`)                                      |
| `opened`      | `opened`      | `boolean` | `true`        | no   | open or expanded state                                                                            |
| `breakpoint`  | `breakpoint`  | `string`  | `"md"`        | no   | breakpoint to switch to mobile overlay (`"sm"`, `"md"`, `"lg"`, `"xl"`, media query, or `"none"`) |
| `shortcut`    | `shortcut`    | `string`  | `"cmd+b"`     | no   | keyboard shortcut chord to toggle sidebar                                                         |
| `persist`     | `persist`     | `string`  | `""`          | no   | localstorage key to remember open and closed state                                                |
| `contained`   | `contained`   | `boolean` | `false`       | no   | contains sidebar within its parent element instead of the viewport                                |

<a name="programmatic-api-d-sidebar"></a>
#### programmatic api

```javascript
const sidebar = select('d-sidebar');

sidebar.open();      // opens or expands the sidebar
sidebar.close();     // closes or collapses the sidebar
sidebar.toggle();    // toggles current state

console.log(sidebar.isOpen);   // true if expanded or mobile open
console.log(sidebar.isMobile); // true if active viewport is below breakpoint
```

<a name="events-d-sidebar"></a>
#### events

| event   | detail                | bubbles | description                 |
| :------ | :-------------------- | :------ | :-------------------------- |
| `open`  | object (`{ mobile }`) | yes     | emitted when sidebar opens  |
| `close` | object (`{ mobile }`) | yes     | emitted when sidebar closes |

<a name="behavior-d-sidebar"></a>
#### behavior

- in icon mode (`collapsible="icon"`), collapsing hides labels and submenus while automatically attaching tooltips to icons.
- clicking `[d-sidebar-rail]` toggles the sidebar; external triggers with `[d-sidebar-trigger]` toggle the sidebar (or targeting by id via `[d-sidebar-trigger="#id"]`).
- below the configured breakpoint, converts to a mobile offcanvas drawer with an interactive backdrop dismiss.

---

### d-skeleton

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/skeleton.mp4" type="video/mp4">
  </video>
</p>

content loading placeholder shapes with customizable dimensions, line counts, and card presets.

```html
<d-skeleton type="text" lines="3" gap="0.5em"></d-skeleton>
<d-skeleton type="circle" size="3em"></d-skeleton>
<d-skeleton type="rect" width="100%" height="150px" radius="0.5em"></d-skeleton>
<d-skeleton type="card"></d-skeleton>
```

#### attributes and properties

| attribute | property | type     | default   | description                                                   |
| :-------- | :------- | :------- | :-------- | :------------------------------------------------------------ |
| `type`    | `type`   | `string` | `"text"`  | preset shape (`"text"`, `"circle"`, `"rect"`, `"card"`)       |
| `lines`   | `lines`  | `number` | `1`       | line count for `"text"` type (last line renders at 60% width) |
| `width`   | `width`  | `string` | `null`    | CSS width                                                     |
| `height`  | `height` | `string` | `null`    | CSS height                                                    |
| `size`    | `size`   | `string` | `null`    | shorthand for equal width and height on circle and rect       |
| `gap`     | `gap`    | `string` | `"0.6em"` | vertical gap between lines in text mode                       |
| `radius`  | `radius` | `string` | `null`    | custom border-radius override                                 |

---

### d-slider

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/slider.mp4" type="video/mp4">
  </video>
</p>

range slider with pointer dragging, keyboard navigation, audio feedback, and automatic indicator text binding.

```html
<d-slider id="volume" name="volume" min="0" max="1" step="0.01" value="0.75"></d-slider>
<span data-slider-volume class="text-content-l">75%</span>
```

#### attributes and properties

| attribute    | property     | type      | default                 | form | description                                             |
| :----------- | :----------- | :-------- | :---------------------- | :--- | :------------------------------------------------------ |
| `value`      | `value`      | `number`  | `0.5`                   | yes  | current numeric slider value                            |
| `min`        | `min`        | `number`  | `0`                     | no   | minimum allowable value                                 |
| `max`        | `max`        | `number`  | `1`                     | no   | maximum allowable value                                 |
| `step`       | `step`       | `number`  | `0.01`                  | no   | step increment for dragging and arrow keys              |
| `name`       | `name`       | `string`  | `""`                    | no   | form payload field name                                 |
| `id`         | `id`         | `string`  | `""`                    | no   | component id (used for `data-slider-{id}` text binding) |
| `disabled`   | `disabled`   | `boolean` | `false`                 | no   | disables slider interactions                            |
| `thumbcolor` | `thumbColor` | `string`  | `"var(--accent)"`       | no   | thumb color variable                                    |
| `fillcolor`  | `fillColor`  | `string`  | `"var(--accent-d)"`     | no   | active track fill color variable                        |
| `trackcolor` | `trackColor` | `string`  | `"var(--border-muted)"` | no   | inactive track background color variable                |
| `sfx`        | `sfx`        | `string`  | embedded                | no   | audio sound effect URL or base64 audio payload          |

#### programmatic api

```javascript
const slider = select('d-slider');

// read current value
const current = slider.getValue();

// set new value (updates visual fill, thumb position, and data-slider indicator)
slider.setValue(0.85);
```

#### events

| event    | detail        | bubbles | description                                                  |
| :------- | :------------ | :------ | :----------------------------------------------------------- |
| `input`  | numeric value | yes     | dispatched during continuous dragging and arrow key movement |
| `change` | numeric value | yes     | dispatched on pointer up or keyboard commit                  |

#### behavior

- supports smooth dragging and keyboard arrow navigation.
- automatically updates text in any element with `data-slider-{id}` with the current percentage value.
- integrates with form submission.

---

### d-text-input

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

#### attributes and properties

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

#### programmatic api

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

#### events

| event    | detail       | bubbles | description                      |
| :------- | :----------- | :------ | :------------------------------- |
| `input`  | string value | yes     | dispatched on keystroke input    |
| `change` | string value | yes     | dispatched on input modification |

#### behavior

- supports global keyboard shortcuts (e.g. `cmd+k`) that focus the field automatically across operating systems.
- displays increment and decrement stepper controls when configured with `type="number"`.

---

### d-toggle

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/toggle.mp4" type="video/mp4">
  </video>
</p>

segmented toggle control with magnetic hover indicators and animated transitions.

```html
<d-toggle name="billing_period" value="0">
  <div default>monthly billing</div>
  <div>annual billing (save 20%)</div>
</d-toggle>
```

#### attributes and properties

| attribute         | property          | type     | default                | form | description                                           |
| :---------------- | :---------------- | :------- | :--------------------- | :--- | :---------------------------------------------------- |
| `value`           | `value`           | `number` | `0`                    | yes  | zero indexed integer of currently selected option     |
| `name`            | `name`            | `string` | `""`                   | no   | form field name                                       |
| `id`              | `id`              | `string` | `""`                   | no   | component id                                          |
| `color`           | `color`           | `string` | `"var(--container-l)"` | no   | container background color variable                   |
| `activecolor`     | `activeColor`     | `string` | `"var(--accent)"`      | no   | active option highlight color                         |
| `textcolor`       | `textColor`       | `string` | `""`                   | no   | inactive option text color (auto contrast by default) |
| `textactivecolor` | `textActiveColor` | `string` | `""`                   | no   | active option text color (auto contrast by default)   |
| `bgcolor`         | `bgColor`         | `string` | `"var(--container-l)"` | no   | base background color                                 |
| `hovercolor`      | `hoverColor`      | `string` | `""`                   | no   | hover background color (theme adaptive by default)    |

#### programmatic api

```javascript
const toggle = select('d-toggle');

// get currently active button element
console.log(toggle.selected);

// activate option by index (number), string index, or button element
toggle.selected = 1;

// read or set numeric value
toggle.value = 0;
```

#### events

| event    | detail                | bubbles | description                              |
| :------- | :-------------------- | :------ | :--------------------------------------- |
| `change` | selected index number | yes     | dispatched when active selection changes |

#### behavior

- renders interactive segmented options with animated hover and selection transitions.
- supports initial selection via the `default` attribute or numeric value.

---

### d-tooltip

popup that displays helpful information when an element receives hover or keyboard focus, built upon the top layer `d-popover` positioning engine with integrated keyboard shortcut glyphs and disabled trigger wrapping.

```html
<d-tooltip text="save document" shortcut="cmd+s" side="top">
  <button class="btn btn-outline">save</button>
</d-tooltip>

<d-tooltip text="action unavailable" side="right">
  <button class="btn btn-outline" disabled>disabled action</button>
</d-tooltip>

<d-tooltip>
  <button slot="trigger" class="btn btn-gray"><i class="dstrn-add-line"></i></button>
  <span slot="content">custom tooltip markup</span>
</d-tooltip>
```

<a name="attributes-and-properties-d-tooltip"></a>
#### attributes and properties

| attribute  | property   | type      | default    | form | description                                                                                           |
| :--------- | :--------- | :-------- | :--------- | :--- | :---------------------------------------------------------------------------------------------------- |
| `text`     | `text`     | `string`  | `""`       | no   | tooltip text content                                                                                  |
| `shortcut` | `shortcut` | `string`  | `""`       | no   | keyboard shortcut combo (e.g. `"cmd+s"`, `"alt+left"`) rendered automatically via `<d-key>`           |
| `side`     | `side`     | `string`  | `"top"`    | no   | preferred placement side (`"top"`, `"bottom"`, `"left"`, `"right"`, `"inline-start"`, `"inline-end"`) |
| `align`    | `align`    | `string`  | `"center"` | no   | alignment along trigger axis (`"start"`, `"center"`, `"end"`)                                         |
| `delay`    | `delay`    | `number`  | `150`      | no   | hover grace delay in milliseconds before reveal                                                       |
| `offset`   | `offset`   | `number`  | `6`        | no   | pixel distance between trigger and tooltip bubble                                                     |
| `disabled` | `disabled` | `boolean` | `false`    | no   | disables tooltip display                                                                              |

<a name="programmatic-api-d-tooltip"></a>
#### programmatic api

```javascript
const tip = select('d-tooltip');

tip.open();
tip.close();
tip.toggle();
tip.setAnchor(select('#custom-anchor'));
const anchor = tip.getAnchor();
tip.text = 'new label';
tip.disabled = true;

// close all active tooltips
dTooltip.closeAll();
```

<a name="events-d-tooltip"></a>
#### events

| event   | detail | bubbles | description                         |
| :------ | :----- | :------ | :---------------------------------- |
| `open`  | none   | yes     | dispatched when the tooltip reveals |
| `close` | none   | yes     | dispatched when the tooltip closes  |

<a name="behavior-d-tooltip"></a>
#### behavior

- renders in the browser top layer via `d-popover`, remaining immune to `overflow: hidden` parent clipping and `z-index` stacking context traps.
- automatically displays formatted platform aware `<d-key>` badges when `shortcut` is provided.
- displays tooltips over disabled buttons by routing pointer events through its outer wrapper.
- integrates with keyboard navigation, revealing on `focusin` and hiding on `focusout`.

---

### d-virtual-list

dom virtualization component skipping layout and paint for offscreen elements using css content visibility and automatic bottom threshold detection.

```html
<d-virtual-list item-size="64px" buffer="800px 0px 800px 0px" threshold="600">
  @foreach(const item of items)
    <div class="p-3 bg-container mb-2 rounded">{{ item.title }}</div>
  @endforeach
</d-virtual-list>
```

<a name="attributes-and-properties-d-virtual-list"></a>
#### attributes and properties

| attribute       | property       | type     | default                 | form | description                                                           |
| :-------------- | :------------- | :------- | :---------------------- | :--- | :-------------------------------------------------------------------- |
| `item-size`     | `itemSize`     | `string` | `"52px"`                | no   | estimated item height fallback for contain intrinsic size             |
| `buffer`        | `buffer`       | `string` | `"600px 0px 600px 0px"` | no   | intersection observer root margin for pre rendering offscreen items   |
| `threshold`     | `threshold`    | `number` | `500`                   | no   | pixel distance to bottom required to fire reachbottom                 |
| `cooldown`      | `cooldown`     | `number` | `250`                   | no   | minimum delay in milliseconds between reachbottom event dispatches    |
| `scroll-parent` | `scrollParent` | `string` | `""`                    | no   | optional css selector targeting custom scroll container               |

<a name="programmatic-api-d-virtual-list"></a>
#### programmatic api

```javascript
const list = select('d-virtual-list');

list.itemSize = '72px';
list.threshold = 400;

// manually trigger bottom proximity calculation
list.checkBottom();
```

<a name="events-d-virtual-list"></a>
#### events

| event         | detail                    | bubbles | description                                                    |
| :------------ | :------------------------ | :------ | :------------------------------------------------------------- |
| `reachbottom` | `{ distance, threshold }` | yes     | dispatched when the user scrolls near the bottom of the list  |

<a name="behavior-d-virtual-list"></a>
#### behavior

- virtualizes offscreen items by applying `content-visibility: hidden` and `contain-intrinsic-size` while keeping light dom nodes intact.
- detects scroll container via parent hierarchy, `scroll-parent` selector, or window viewport.
- uses an anchor element and intersection observer to reliably detect bottom proximity and emit `reachbottom` events for infinite scrolling.
- clears and pauses bottom checks when offscreen.

---

<a name="overriding-built-in-components"></a>

## overriding built in components

every built in component behavior can be customized, extended, or completely overwritten by application code. to override a built in component, place a file with the exact same component filename under `public/js/components/` (such as `public/js/components/dCheckbox.js`).

when compiling the frontend bundle, the compiler automatically detects the user file and replaces the built in component implementation in place.

```javascript
// public/js/components/dCheckbox.js
class dCheckbox extends dComponent {
  static form = true;
  static get tag() { return 'd-checkbox'; }

  // custom template or methods
}

dComponent.define(dCheckbox);
```

<a name="immutable-base-class"></a>

### immutable base class

all custom elements and overrides extend `dComponent`. unlike UI components, `dComponent` is the immutable framework base class providing reactivity, reflection, and lifecycle management. the compiler always preserves the core `dComponent` implementation at the root of the bundle so all components inherit from it consistently.

---

<a name="dcomponent-base-class"></a>

## dComponent (base class)

`dComponent` is the authoring base class for all custom web components in dframework. it provides property reflection, reactive proxy state, form integration, surgical dom rendering, and automatic memory cleanup.

```javascript
class dCounter extends dComponent {
  static tag = 'd-counter';
  static form = true; // opt into native form submission integration

  static props = {
    value: { type: 'number', default: 0, form: true },
    step: { type: 'number', default: 1 },
    disabled: { type: 'boolean', default: false },
    label: { type: 'string', default: 'count' }
  };

  template() {
    // called once on mount to establish initial innerHTML
    return `
      <span class="label">${this.label}</span>
      <button class="dec" type="button">−</button>
      <span class="value">${this.value}</span>
      <button class="inc" type="button">+</button>
    `;
  }

  mount() {
    // called once after template initialization
    this.effect(() => {
      // reading this.state or props registers auto dependencies
      document.title = `count: ${this.state.value ?? this.value}`;
      return () => { document.title = 'app'; }; // optional cleanup
    });
  }

  render() {
    // called automatically on prop or state changes
    // use for surgical dom updates only (never mutate this.state here)
    this.refs('.value')[0].textContent = this.state.value ?? this.value;
    this.refs('button.dec')[0].disabled = this.disabled;
    this.refs('button.inc')[0].disabled = this.disabled;

    // listeners attached inside render() are cleared and rebound automatically
    this.listen(this.refs('button.dec')[0], 'click', () => this.decrement());
    this.listen(this.refs('button.inc')[0], 'click', () => this.increment());
  }

  onPropChanged(name, oldVal, newVal) {
    // called on attribute or property modifications
  }

  increment() {
    this.setState(s => { s.value = (s.value ?? this.value) + this.step; });
  }

  decrement() {
    this.setState(s => { s.value = (s.value ?? this.value) - this.step; });
  }
}

dComponent.define(dCounter);
```

<a name="component-definition-and-registration"></a>

### component definition and registration

register components using `dComponent.define(Class)`:

```javascript
dComponent.define(dCounter);
```

- automatically defines getter and setter properties on the prototype with type coercion (`number`, `boolean`, `string`, `object`, `array`).
- binds observed attributes to property setters.
- registers with `customElements.define` using `Class.tag`.

<a name="lifecycle-methods"></a>

### lifecycle methods

| method                          | timing               | description                                           |
| :------------------------------ | :------------------- | :---------------------------------------------------- |
| `template()`                    | before mount         | returns initial HTML string to populate inner dom     |
| `mount()`                       | on first connection  | initializes component logic and registers effects     |
| `mountSync()`                   | immediate/manual     | forces immediate synchronous mount without RAF lag    |
| `connected()`                   | on every connection  | called whenever element is attached to document       |
| `render()`                      | on state/prop change | performs surgical dom node updates                    |
| `onPropChanged(name, old, new)` | on prop mutation     | reacts to specific property changes                   |
| `syncForm()`                    | form/manual          | synchronizes ElementInternals form value; can be overridden for custom form submission rules |
| `onscreen()`                    | on viewport entry    | called when component enters viewport                 |
| `offscreen()`                   | on viewport exit     | called when component leaves viewport                 |
| `destroy()`                     | on disconnection     | teardown hook called when element is removed from dom |

> [!NOTE]
> `mount()` initialization is batched via `requestAnimationFrame` upon initial dom connection. this guarantees that all child nodes parsed in the light dom are present and available before `mount()` executes. when programmatically creating child custom elements or when synchronous initialization is required immediately after instantiation, call `element.mountSync()` or `dComponent.mountSync(element)`. for template components, template markup is preparsed on definition and stamped via cached `cloneNode(true)`. subsequent property and state updates are batched into a static 3 phase microtask flush.

<a name="fragment-caching-and-batch-mounting"></a>

### fragment caching and batch mounting

for maximum throughput and xss safety when generating dynamic content, setting inner markup, or mounting lists of components, `dComponent` provides template caching utilities and safe dom replacement:

```javascript
// parse html into a cached template fragment (avoids repeated innerHTML string parsing)
const frag = dComponent.fragment('<div class="list-item"><span>title</span></div>');

// append multiple nodes, fragments, or html strings in a single dom operation
dComponent.appendMany(container, frag1, frag2, '<div class="extra">item</div>');

// safely set or replace element children using cached template fragments or nodes
this.setChildren('<div class="card"><span class="title"></span></div>');
```

<a name="rendering-strategy-and-surgical-updates"></a>

### rendering strategy and surgical updates

`dComponent` avoids virtual dom overhead. dom nodes are created once in `template()` or `mount()`, while dynamic updates are applied surgically inside `render()`.

> [!IMPORTANT]
> `this.state` must never be modified inside `render()`. state mutations inside `render()` are blocked to prevent infinite update loops.

<a name="batched-microtask-update-queue"></a>

### batched microtask update queue

property modifications and state mutations are automatically coalesced across all active components into a static 3 phase microtask flush:

1. **phase 1 (attribute sync):** synchronizes observed html attributes for changed properties.
2. **phase 2 (rendering):** clears transient render listeners and executes `render()` once per updated component.
3. **phase 3 (effects & form sync):** reevaluates reactive effects, invalidates refs caches, synchronizes form internals, and dispatches an aggregated debugbar event.

```javascript
// multiple synchronous state mutations trigger exactly one render cycle in the next microtask
this.state.title = 'new title';
this.state.count = 42;
```

<a name="reactive-state"></a>

### reactive state

```javascript
// update state and trigger microtask batched render()
this.setState({ active: true });

// functional updater
this.setState(s => { s.count++; s.updatedAt = Date.now(); });

// update state without triggering render() (still triggers effect loops)
this.setState({ timer: Date.now() }, false);
```

<a name="effects-and-dependency-tracking"></a>

### effects and dependency tracking

```javascript
// auto tracking effect (tracks any accessed this.state or prop)
this.effect(() => {
  console.log('value updated:', this.state.value);
  return () => console.log('cleaning up effect');
});

// explicit dependency array effect
this.effect(() => {
  fetchData(this.query);
}, () => [this.query]);
```

<a name="automatic-cleanup-apis"></a>

### automatic cleanup apis

these methods automatically clean up listeners, timers, and animation frame requests when the component is removed from the dom:

```javascript
// event listener returning unlistener function
const off = this.listen(element, 'click', (e) => this.handleClick(e));
off(); // manual cleanup if needed

// listen to multiple elements returning composite unlistener function
const offAll = this.listenAll(elements, 'click', (e) => this.handleClick(e));
offAll(); // detaches all listeners

// unlisten manually if needed
this.unlisten(element, 'click', handler);

// auto cleared timers
this.setTimeout(() => this.tick(), 1000);
this.setInterval(() => this.refresh(), 5000);

// auto cleared animation frame
this.requestAnimationFrame((time) => this.draw(time));
```

<a name="dom-caching-and-refs"></a>

### dom caching and refs

`this.refs(selector)` caches queried element arrays between renders to eliminate repeated dom query overhead:

```javascript
const [inputEl] = this.refs('input.search');
```

<a name="inner-content-capture"></a>

### inner content capture

access original light dom nodes passed into the custom element before `template()` execution:

```javascript
this.originalChildren // array of cloned child nodes
this.originalHTML     // original innerHTML string
this.cloneChildren()  // returns fresh clones of original child nodes
```

<a name="form-integration"></a>

### form integration

when `static form = true`, `dComponent` integrates with parent `<form>` elements and submits properties marked with `form: true`:

```javascript
static form = true;
static props = {
  value: { type: 'string', default: '', form: true }
};

// access parent form element
console.log(this.form);
```

supports single values, file instances (`File`), file arrays (`File[]`), and multiple form properties automatically.

to implement custom form submission behavior (such as omitting unchecked checkbox or radio values, or clearing values when disabled), override `syncForm()`:

```javascript
syncForm() {
  if (!this._internals) return;
  if (this.checked && !this.disabled) {
    this._internals.setFormValue(this.value || 'on');
  } else {
    this._internals.setFormValue(null);
  }
}
```

<a name="attribute-reflection"></a>

### attribute reflection and programmatic updates

to reflect property values to the host DOM element without triggering recursive `attributeChangedCallback` executions, use `this.reflectAttribute(name, value)`:

```javascript
// immediately syncs attribute on host element safely
this.reflectAttribute('placement', 'before');
```

to check if an attribute or property update originated from a scripted property mutation or `onPropChanged()`, inspect `this.isProgrammaticUpdate`:

```javascript
if (this.isProgrammaticUpdate) return;
```

<a name="internal-event-emitter"></a>

### internal event emitter

```javascript
// listen to component events returning unlistener function
const off = this.on('custom-event', (data) => console.log(data));
off(); // detach listener

// remove listener by callback reference
this.off('custom-event', handler);

// emit event to internal listeners and dispatch bubbling composed customevent
this.emit('custom-event', { id: 123 });
```

<a name="websocket-integration"></a>

### websocket integration

```javascript
// emit event through global websocket connection
this.wire('chat:send', { message: 'hello' });
```
