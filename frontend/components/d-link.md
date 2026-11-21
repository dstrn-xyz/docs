# d-link

client side navigation link component and directive that intercepts clicks, fetches content asynchronously, and performs dom swaps without full page reloads.

```html
<!-- custom element -->
<d-link href="/dashboard" class="nav-link">dashboard</d-link>

<!-- anchor directive alias -->
<a d-link href="/settings" target="#settings-panel" mode="inner">settings</a>
```

<a name="attributes-and-properties-d-link"></a>

## attributes and properties

| attribute         | property         | type      | default   | form | description                                                            |
| :---------------- | :--------------- | :-------- | :-------- | :--- | :--------------------------------------------------------------------- |
| `href`            | `href`           | `string`  | `""`      | no   | target destination url                                                 |
| `target`          | `target`         | `string`  | `""`      | no   | comma separated css selectors of elements to swap in current page      |
| `src`             | `src`            | `string`  | `""`      | no   | comma separated css selectors of elements to extract from fetched page |
| `mode`            | `mode`           | `string`  | `"inner"` | no   | content swap mode (`"inner"`, `"outer"`, `"append"`, `"prepend"`)      |
| `preserve-scroll` | `preserveScroll` | `boolean` | `false`   | no   | preserves window scroll position across navigation                     |
| `d-full-reload`   | `dFullReload`    | `boolean` | `false`   | no   | bypasses client side router and triggers standard browser reload       |

<a name="programmatic-api-d-link"></a>

## programmatic api

```javascript
// navigate programmatically using dSPA
await dSPA.navigate('/dashboard', {
  targets: ['#content'],
  mode: 'inner',
  preserveScroll: true
});
```

<a name="behavior-d-link"></a>

## behavior

- intercepts click events on `<d-link>` elements and `<a d-link>` anchors.
- ignores clicks with modifier keys (cmd, ctrl, shift, alt), target `_blank`, hash links, or external protocol urls (mailto, tel, javascript).
- validates same origin before initiating fetch requests.
- falls back to full browser navigation if network fetch fails.
