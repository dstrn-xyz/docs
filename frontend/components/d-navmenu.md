# d-navmenu

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
## attributes and properties

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
## programmatic api

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
## events

| event   | detail                   | bubbles | description                           |
| :------ | :----------------------- | :------ | :------------------------------------ |
| `open`  | `{ index, title, item }` | yes     | dispatched when a submenu opens       |
| `close` | none                     | yes     | dispatched when active submenu closes |

<a name="behavior-and-d-hamburger-integration"></a>
## behavior and d-hamburger integration

- shared morphing viewport panel animates width and height while cross sliding contents between navigation tabs.
- when `<d-hamburger target="#nav-id" breakpoint="md">` targets `<d-navmenu id="nav-id">`, the desktop navigation bar automatically hides on mobile viewports and converts all direct links and `[d-submenu]` sections into drilldown mobile drawer panels with no duplicate markup.
- full keyboard navigation (`ArrowLeft` / `ArrowRight` between triggers, `ArrowDown` / `Enter` to focus into active panel, `Escape` to close).
