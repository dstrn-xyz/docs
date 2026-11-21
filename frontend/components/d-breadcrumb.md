# d-breadcrumb

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
## attributes and properties

| attribute   | property    | type     | default | form | description                                                                         |
| :---------- | :---------- | :------- | :------ | :--- | :---------------------------------------------------------------------------------- |
| `separator` | `separator` | `string` | `""`    | no   | custom text (e.g. `"/"`) or icon class (`"dstrn-right-line"`); defaults to chevron  |
| `max-items` | `maxItems`  | `number` | `0`     | no   | collapses middle items into an interactive ellipsis button when count exceeds limit |
| `items`     | `items`     | `array`  | `null`  | no   | optional array of `{ label, href, link }` objects overriding light DOM children     |

<a name="programmatic-api-d-breadcrumb"></a>
## programmatic api

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
## events

| event      | detail | bubbles | description                                           |
| :--------- | :----- | :------ | :---------------------------------------------------- |
| `expand`   | none   | yes     | dispatched when collapsed middle items are expanded   |
| `collapse` | none   | yes     | dispatched when `collapse()` restores collapsed state |

<a name="behavior-d-breadcrumb"></a>
## behavior

- preserves live child DOM nodes (`<a d-link>`, `<d-link>`, buttons, or nested popover triggers) without reparsing html.
- automatically highlights the final breadcrumb item as the active page and mirrors directional separator icons in RTL layouts.
- clicking the collapsed ellipsis button opens a floating popover containing the hidden middle links.
