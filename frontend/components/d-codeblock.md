# d-codeblock

syntax highlighted code block component with a copy to clipboard button, filename header bar, language badge, line numbers, line highlighting, and diff mode.

```html
<!-- standard code block with title and language -->
<d-codeblock title="Server.js" lang="javascript" copyable>
const port = 3000;
function listen() {
  console.log("listening on", port);
}
</d-codeblock>

<!-- line highlighting and line numbers -->
<d-codeblock lang="javascript" highlight="2,4-5">
const a = 1;
const b = 2;
const c = 3;
const sum = a + b + c;
console.log(sum);
</d-codeblock>

<!-- diff mode with floating copy button -->
<d-codeblock diff copyable>
 initial config
-const timeout = 1000;
+const timeout = 5000;
 return timeout;
</d-codeblock>
```

<a name="attributes-and-properties-d-codeblock"></a>
## attributes and properties

| attribute      | property      | type      | default | form | description                                                           |
| :------------- | :------------ | :-------- | :------ | :--- | :-------------------------------------------------------------------- |
| `title`        | `title`       | `string`  | `""`    | no   | filename or label displayed in the header bar                         |
| `lang`         | `lang`        | `string`  | `""`    | no   | code language identifier displayed as an uppercase badge              |
| `code`         | `code`        | `string`  | `""`    | no   | programmatic code string input                                        |
| `line-numbers` | `lineNumbers` | `boolean` | `true`  | no   | whether to render line numbers beside each row                        |
| `highlight`    | `highlight`   | `string`  | `""`    | no   | comma separated line numbers and ranges to highlight (e.g. `"2,4-6"`) |
| `diff`         | `diff`        | `boolean` | `false` | no   | enables diff styling for lines starting with `+` or `-`               |
| `copyable`     | `copyable`    | `boolean` | `true`  | no   | whether to display a copy button (in header or floating top right)    |
| `wrap`         | `wrap`        | `boolean` | `false` | no   | wraps long lines instead of horizontal scrolling                      |
| `max-height`   | `maxHeight`   | `string`  | `""`    | no   | maximum scrollable height for long code snippets                      |
| `code-font-size` | `codeFontSize` | `string` | `""`   | no   | custom font size applied specifically to code text                    |

<a name="programmatic-api-d-codeblock"></a>
## programmatic api

```javascript
const block = select('d-codeblock');

// trigger copy programmatically
block.copy();

// read plain code text
block.codeText; // string
```

<a name="events-d-codeblock"></a>
## events

| event  | detail             | bubbles | description                                     |
| :----- | :----------------- | :------ | :---------------------------------------------- |
| `copy` | `{ text: string }` | yes     | dispatched when code is copied to the clipboard |

<a name="behavior-d-codeblock"></a>
## behavior

- when `title` or `lang` is provided, displays an integrated header bar with file icon, filename, language badge, and copy button.
- when neither `title` nor `lang` is provided, renders a copy button anchored in the top right corner.
- animated copy action transitions the copy icon into a green checkmark, reverting after 2 seconds.
