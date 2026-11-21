# d-modal

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

## attributes and properties

| attribute    | property     | type      | default | form | description                                                           |
| :----------- | :----------- | :-------- | :------ | :--- | :-------------------------------------------------------------------- |
| `opened`     | `opened`     | `boolean` | `false` | no   | whether the modal is currently visible                                |
| `persistent` | `persistent` | `boolean` | `false` | no   | whether the element remains in the dom when closed via backdrop click |

## programmatic api

```javascript
const modal = select('d-modal');

// open modal (locks body scroll)
modal.open();

// close modal (runs exit animation and unlocks body scroll)
modal.close();

// close, await exit transition, and remove element from dom
await modal.destroy();
```

## events

| event   | detail | bubbles | description                  |
| :------ | :----- | :------ | :--------------------------- |
| `open`  | none   | yes     | dispatched when modal opens  |
| `close` | none   | yes     | dispatched when modal closes |

## behavior and styling boundaries

- displays an overlay dialog and prevents page scrolling while visible.
- when `persistent` is omitted (or `false`), clicking the backdrop closes and destroys the element (`destroy()`), removing it from the dom.
- when `persistent` is present (`<d-modal persistent>`), clicking the backdrop closes the dialog (`close()`) without removing the element from the dom, allowing it to be reopened.
- `<d-modal>` automatically injects `.d-modal-wrapper` (which provides `var(--container-l)` background, border, radius and shadow) and `.d-modal-content` (flex container with scroll management).
- to customize or restyle the modal appearance, target `d-modal .d-modal-wrapper` or `d-modal .d-modal-content` in your css.
