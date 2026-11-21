# d-file-input

<p align="center">
  <video autoplay loop muted playsinline width="400">
    <source src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.videos/file-input.mp4" type="video/mp4">
  </video>
</p>

drag and drop file upload container with interactive item list, formatted byte size calculations, and compact bar modes.

```html
<d-file-input accept=".pdf,.zip,.csv" name="documents" multiple compact></d-file-input>
```

## attributes and properties

| attribute     | property      | type             | default                | form | description                                  |
| :------------ | :------------ | :--------------- | :--------------------- | :--- | :------------------------------------------- |
| `accept`      | `accept`      | `string`         | `"*/*"`                | no   | allowed mime types or file extensions        |
| `icon`        | `icon`        | `string`         | `"dstrn-folder-line"`  | no   | icon class displayed in dropzone             |
| `placeholder` | `placeholder` | `string`         | `"drag & drop file"`   | no   | primary dropzone title                       |
| `subtitle`    | `subtitle`    | `string`         | `"or click to browse"` | no   | secondary helper subtitle                    |
| `compact`     | `compact`     | `boolean`        | `true`                 | no   | renders as a slim horizontal bar when true   |
| `multiple`    | `multiple`    | `boolean`        | `false`                | no   | allows selecting and dropping multiple files |
| `name`        | `name`        | `string`         | `""`                   | no   | form payload field name                      |
| `id`          | `id`          | `string`         | `""`                   | no   | component id                                 |
| `value`       | `value`       | `object`/`array` | `null`                 | yes  | `File`, `File[]`, or url string payload      |

## programmatic api

```javascript
const fileInput = select('d-file-input');

// reset dropzone, clearing value, file list, and native input
fileInput.reset();

// inspect selected files
console.log(fileInput.value); // File instance or File[] array
```

## events

| event    | detail                      | bubbles | description                                             |
| :------- | :-------------------------- | :------ | :------------------------------------------------------ |
| `change` | `File`, `File[]`, or `null` | yes     | dispatched when files are selected, dropped, or deleted |

## behavior

- drag and drop file upload with file list display, formatted sizes (Bytes, KB, MB, GB), and delete buttons.
- automatically clears error states when a valid file is dropped.
- integrates with standard form submission.
