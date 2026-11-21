# d-image-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/image-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop image dropzone with instant client preview and action overlay.

```html
<d-image-input accept="image/png,image/jpeg" name="avatar" fit="cover" placeholder="drop profile photo"></d-image-input>
```

## attributes and properties

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

## programmatic api

```javascript
const imgInput = select('d-image-input');

// reset dropzone and clear image preview
imgInput.reset();

// get or set preview src directly
imgInput.preview = '/img/avatar.png';

// read selected File instance
console.log(imgInput.value);
```

## events

| event    | detail           | bubbles | description                                               |
| :------- | :--------------- | :------ | :-------------------------------------------------------- |
| `change` | `File` or `null` | yes     | dispatched when an image is selected, dropped, or deleted |

## behavior

- provides drag and drop image selection with instant preview and action buttons to replace or remove images.
- automatically integrates with form submission and clears error states on upload.
