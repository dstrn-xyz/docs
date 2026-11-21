# d-menubar

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
## attributes and properties

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
## programmatic api

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
## events

| event    | detail                                    | bubbles | description                                             |
| :------- | :---------------------------------------- | :------ | :------------------------------------------------------ |
| `select` | `{ value, item, menu }`                   | yes     | dispatched when an action item is invoked               |
| `change` | `{ value, checked }` or `{ name, value }` | yes     | dispatched when a check item or radio selection changes |
| `open`   | `{ menu, index }`                         | yes     | dispatched when a top level menu panel opens            |
| `close`  | `{ menu, index }`                         | yes     | dispatched when a menu panel closes                     |

<a name="behavior-d-menubar"></a>
## behavior

- renders dropdown panels in the browser top layer via `d-popover`, avoiding clipping inside nested overflow containers.
- clicking any top level menu opens the panel and activates menu mode; moving the cursor across sibling top triggers switches between menus.
- submenus reveal on pointer hover with grace delay and support nested hierarchies.
- supports complete keyboard navigation (`ArrowLeft` / `ArrowRight` between top triggers, `ArrowDown` / `ArrowUp` through menu items, `ArrowRight` / `ArrowLeft` into and out of submenus, `Enter` / `Space` to invoke, `Escape` to close).
