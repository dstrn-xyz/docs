# d-dropdown

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

## attributes and properties

| attribute | property | type      | default      | form | description                                      |
| :-------- | :------- | :-------- | :----------- | :--- | :----------------------------------------------- |
| `header`  | `header` | `string`  | `"dropdown"` | no   | title text displayed on the clickable header bar |
| `opened`  | `opened` | `boolean` | `false`      | no   | whether the dropdown is expanded                 |

## programmatic api

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

## events

| event   | detail | bubbles | description                               |
| :------ | :----- | :------ | :---------------------------------------- |
| `open`  | none   | yes     | dispatched when the dropdown panel opens  |
| `close` | none   | yes     | dispatched when the dropdown panel closes |

## behavior

- clicking the header expands or collapses child content with an animated chevron indicator.
- dynamically updates header text and open state when properties change.
