# frontend components

dframework ships a complete library of zero-dependency custom web components compiled into the frontend bundle. components are standard custom elements with no virtual dom, no compiler requirements, and no client build step. the framework compiles and bundles component files in `public/js/components/` automatically on server boot.

built in components support native client side localization out of the box with english and japanese dictionaries and automatic tag scoped resolution (`this.t()`). for a guide on switching locales, customizing strings, or translating custom elements, see the [localization guide](../other/localization.md#dcomponent-localization).

## component catalog

components are split into focused individual documentation files with anatomy, live usage examples, attributes, programmatic apis, events, and keyboard behaviors.

### layout and navigation

- [`d-sidebar`](./components/d-sidebar.md): application shell sidebar with offcanvas sliding, icon collapse mode, and mobile drawers.
- [`d-navmenu`](./components/d-navmenu.md): responsive navigation menu with animated indicator pills, viewports, and mobile hamburger integration.
- [`d-breadcrumb`](./components/d-breadcrumb.md): hierarchical navigation trail with middle item collapsing and custom separators.
- [`d-drawer`](./components/d-drawer.md): sliding offcanvas overlay drawer with gesture dismiss and edge attachment.
- [`d-hamburger`](./components/d-hamburger.md): responsive mobile drawer menu with automatic drilldown panels.
- [`d-menubar`](./components/d-menubar.md): desktop application menu bar with keyboard arrow navigation.
- [`d-link`](./components/d-link.md): declarative client side router link with morphing transitions.

### forms and inputs

- [`d-text-input`](./components/d-text-input.md): text field with clear button, floating labels, and form integration.
- [`d-combobox`](./components/d-combobox.md): searchable select filter with remote querying and keyboard navigation.
- [`d-toggle`](./components/d-toggle.md): segmented option switcher with sliding active indicator pill.
- [`d-checkbox`](./components/d-checkbox.md): tri-state accessible checkbox with animated checkmark.
- [`d-radiobutton`](./components/d-radiobutton.md): single select radio button with keyboard group navigation.
- [`d-datepicker`](./components/d-datepicker.md): calendar input field with date formatting and popup picker.
- [`d-calendar`](./components/d-calendar.md): interactive date grid supporting single, range, and multi-selection.
- [`d-color-picker`](./components/d-color-picker.md): visual color picker with hex, rgb, and hsl inputs.
- [`d-file-input`](./components/d-file-input.md): drag and drop file upload field with progress indicators.
- [`d-image-input`](./components/d-image-input.md): image upload field with preview and aspect ratio constraints.
- [`d-slider`](./components/d-slider.md): numeric range slider with tooltip bubble and step snapping.
- [`d-otp`](./components/d-otp.md): one-time verification code input with auto-advance and clipboard paste.
- [`d-form`](./components/d-form.md): reactive ajax form wrapper with automatic error handling and loading state.

### overlays and feedback

- [`d-modal`](./components/d-modal.md): dialog overlay with backdrop dismiss, escape key handling, and animations.
- [`d-popover`](./components/d-popover.md): anchored floating content container with smart viewport positioning.
- [`d-notification`](./components/d-notification.md): toast notification system with stacking and auto-dismiss timer.
- [`d-tooltip`](./components/d-tooltip.md): hover and focus tooltip with keyboard shortcut badges.
- [`d-context-menu`](./components/d-context-menu.md): right-click context menu with nested submenus and positioning.
- [`d-dropdown`](./components/d-dropdown.md): dropdown trigger and action menu with keyboard selection.

### data and display

- [`d-codeblock`](./components/d-codeblock.md): syntax highlighted code display with line numbers and copy button.
- [`d-skeleton`](./components/d-skeleton.md): content loading placeholder shapes with customizable dimensions.
- [`d-progress`](./components/d-progress.md): determinate and indeterminate progress bars with value labels.
- [`d-virtual-list`](./components/d-virtual-list.md): high performance virtualized scroller for large datasets.
- [`d-key`](./components/d-key.md): keyboard shortcut badge with platform specific symbol conversion.
- [`d-loader`](./components/d-loader.md): spinner and pulse animation loader for asynchronous states.

### actions and controls

- [`d-buttongroup`](./components/d-buttongroup.md): fused button toolbar container with shared border radius.
- [`d-hold-button`](./components/d-hold-button.md): long press confirmation button with animated fill ring.
- [`d-icon-button`](./components/d-icon-button.md): accessible square button optimized for icon fonts.
- [`d-morph`](./components/d-morph.md): animated shape and layout transition container.
- [`d-questionnaire`](./components/d-questionnaire.md): multi-step interactive wizard with form validation.

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

