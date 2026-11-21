# d-sidebar

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
## attributes and properties

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
## programmatic api

```javascript
const sidebar = select('d-sidebar');

sidebar.open();      // opens or expands the sidebar
sidebar.close();     // closes or collapses the sidebar
sidebar.toggle();    // toggles current state

console.log(sidebar.isOpen);   // true if expanded or mobile open
console.log(sidebar.isMobile); // true if active viewport is below breakpoint
```

<a name="events-d-sidebar"></a>
## events

| event   | detail                | bubbles | description                 |
| :------ | :-------------------- | :------ | :-------------------------- |
| `open`  | object (`{ mobile }`) | yes     | emitted when sidebar opens  |
| `close` | object (`{ mobile }`) | yes     | emitted when sidebar closes |

<a name="behavior-d-sidebar"></a>
## behavior

- in icon mode (`collapsible="icon"`), collapsing hides labels and submenus while automatically attaching tooltips to icons.
- clicking `[d-sidebar-rail]` toggles the sidebar; external triggers with `[d-sidebar-trigger]` toggle the sidebar (or targeting by id via `[d-sidebar-trigger="#id"]`).
- below the configured breakpoint, converts to a mobile offcanvas drawer with an interactive backdrop dismiss.
