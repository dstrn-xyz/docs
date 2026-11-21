# d-context-menu

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

## attributes and properties

| attribute | property | type      | default | form | description                                                     |
| :-------- | :------- | :-------- | :------ | :--- | :-------------------------------------------------------------- |
| `opened`  | `opened` | `boolean` | `false` | no   | whether the context menu is open                                |
| `parent`  | `parent` | `string`  | `""`    | no   | selector for target parent element (defaults to parent element) |

## programmatic api

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

## events

| event   | detail          | bubbles | description                                                |
| :------ | :-------------- | :------ | :--------------------------------------------------------- |
| `click` | clicked element | yes     | emitted through component emitter when an item is selected |
| `open`  | none            | yes     | dispatched when context menu opens                         |
| `close` | none            | yes     | dispatched when context menu closes                        |

## behavior

- opens on right click within its parent container and suppresses the browser default context menu.
- calculates menu dimensions and automatically bounds coordinates so the menu never overflows parent or screen edges.
- clicking outside closes the menu immediately without triggering click actions or links on underlying elements.
- automatically closes any other open context menus before opening.
- when creating dynamically in javascript, listen to `'click'` directly on the component element; item wrappers are created during component mount in the next animation frame.
