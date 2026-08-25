# frontend utilities

- [frontend utilities](#frontend-utilities)
  - [introduction](#introduction)
  - [button classes](#button-classes)
  - [responsive prefixes](#responsive-prefixes)
  - [layout](#layout)
    - [positioning](#positioning)
    - [grid](#grid)
    - [flexbox](#flexbox)
  - [spacing](#spacing)
  - [sizing](#sizing)
  - [typography](#typography)
  - [appearance](#appearance)
  - [interaction](#interaction)
  - [transitions & animation](#transitions--animation)
  - [performance](#performance)
  - [dom functions](#dom-functions)
    - [selection](#selection)
    - [events](#events)
    - [classes](#classes)
    - [timing](#timing)
  - [data functions](#data-functions)
    - [escaping](#escaping)
    - [type checking](#type-checking)
    - [array utilities](#array-utilities)
    - [extras](#extras)
  - [notify & modal](#notify--modal)
    - [notify](#notify)
    - [modal](#modal)

<a name="introduction"></a>

## introduction

dframework ships a complete utility layer for styling and dom interaction. utility classes are generated at compile time. no runtime css in js. frontend functions are available globally in every page script and component.

<a name="button-classes"></a>

## button classes

<p align="center">
  <img width="400" src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.images/buttons.png"></img>
</p>

you can use prestyled buttons by using the base class `btn` and then chaining a prefix class `btn-*` where `*` represents the button color.

| available colors          | description          |
| -------------- | -------------------- |
| `.btn-accent` | accent colored button        |
| `.btn-red`      | soft red button        |
| `.btn-yellow`   | soft yellow button       |
| `.btn-green`   | soft green button        |
| `.btn-blue`   | soft blue button |
| `.btn-purple`   | soft purple button |
| `.btn-pink`   | soft pink button |
| `.btn-gray`   | `var(--container)` colored button |
| `.btn-transparent`   | transparent button (use for animation) |

<a name="responsive-prefixes"></a>

## responsive prefixes

any utility class can be prefixed with a viewport breakpoint. prefixed classes apply only when the viewport meets or exceeds the defined width.

| prefix   | min-width |
| -------- | --------- |
| `sm:`    | `480px`   |
| `md:`    | `768px`   |
| `lg:`    | `1024px`  |
| `xl:`    | `1280px`  |
| `xxl:`   | `1536px`  |
| `ultra:` | `1920px`  |

```html
<div class="flex-column md:flex-row g-1 md:g-2">...</div>
```

<a name="layout"></a>

## layout

### positioning

| class                    | description                                                      |
| ------------------------ | ---------------------------------------------------------------- |
| `.relative`              | `position: relative`                                             |
| `.fixed`                 | `position: fixed`                                                |
| `.sticky`                | `position: sticky`                                               |
| `.absolute`              | `position: absolute`                                             |
| `.absolute-center`       | absolute center via translate                                    |
| `.absolute-x-center`     | absolute horizontal center                                       |
| `.absolute-y-center`     | absolute vertical center                                         |
| `.absolute-fill`         | fill parent element absolutely                                   |
| `.top-0` / `.bottom-0`   | `top: 0` / `bottom: 0`                                           |
| `.left-0` / `.right-0`   | `left: 0` / `right: 0`                                           |
| `.top-50` / `.bottom-50` | `top: 50%` / `bottom: 50%`                                       |
| `.left-50` / `.right-50` | `left: 50%` / `right: 50%`                                       |
| `.top-[1–10]`            | `top` in em steps                                                |
| `.bottom-[1–10]`         | `bottom` in em steps                                             |
| `.left-[1–10]`           | `left` in em steps                                               |
| `.right-[1–10]`          | `right` in em steps                                              |
| `.inset-0`               | `top/right/bottom/left: 0` (no position set)                     |
| `.z-[0–auto]`            | z-index: `0`, `1`, `2`, `3`, `4`, `5`, `10`, `50`, `100`, `auto` |

### grid

| class                      | description                   |
| -------------------------- | ----------------------------- |
| `.grid-center`             | centered grid                 |
| `.grid-2`                  | 2-column grid                 |
| `.grid-3`                  | 3-column grid                 |
| `.grid-4`                  | 4-column grid                 |
| `.grid-auto`               | dense auto-flow grid          |
| `.col-span-[1–4,full]`     | grid column span              |
| `.row-span-[1-3,full]`     | grid row span                 |
| `.col-start-[1–4]`         | grid column start             |
| `.col-end-[1–4]`           | grid column end               |
| `.row-start-[1–3]`         | grid row start                |
| `.row-end-[1–3]`           | grid row end                  |

### flexbox

| class                                               | description                   |
| --------------------------------------------------- | ----------------------------- |
| `.flex-row`                                         | `flex-direction: row`         |
| `.flex-column`                                      | `flex-direction: column`      |
| `.flex-center`                                      | center on both axes           |
| `.flex-wrap`                                        | `flex-wrap: wrap`             |
| `.flex-1`                                           | `flex: 1 1 0%`                |
| `.flex-auto`                                        | `flex: 1 1 auto`              |
| `.flex-none`                                        | `flex: none`                  |
| `.grow` / `.grow-0`                                 | individual flex-grow control  |
| `.shrink` / `.shrink-0`                             | individual flex-shrink control |
| `.order-[1–5]` / `.order-[first|last|none]`         | flex/grid item order          |
| `.justify-[start|end|center|between|around|evenly]` | main axis alignment           |
| `.align-[start|end|center|baseline|stretch]`        | cross axis alignment          |
| `.self-[start|end|center|stretch|baseline]`         | per item cross axis alignment |

<a name="spacing"></a>

## spacing

values available for all spacing classes: `0`, `025`, `05`, `075`, `1`, `125`, `15`, `175`, `2` through `10` (integer steps). margin also accepts `auto`.

| prefix          | applies to           |
| --------------- | -------------------- |
| `.p-`           | all padding          |
| `.px-`          | horizontal padding   |
| `.py-`          | vertical padding     |
| `.pt-` / `.pb-` | top / bottom padding |
| `.pl-` / `.pr-` | left / right padding |
| `.m-`           | all margin           |
| `.mx-`          | horizontal margin    |
| `.my-`          | vertical margin      |
| `.mt-` / `.mb-` | top / bottom margin  |
| `.ml-` / `.mr-` | left / right margin  |
| `.g-`           | gap (both axes)      |
| `.gx-`          | column gap only      |
| `.gy-`          | row gap only         |

<a name="sizing"></a>

## sizing

| prefix                | values                                                    |
| --------------------- | --------------------------------------------------------- |
| `.w-`                 | `100vw`, `5`–`100` (5 step), `auto`, `min`, `max`, `fit`  |
| `.h-`                 | `100svh`, `5`–`100` (5 step), `auto`, `min`, `max`, `fit` |
| `.min-w-` / `.max-w-` | same as `.w-`                                             |
| `.min-h-` / `.max-h-` | same as `.h-`                                             |

<a name="typography"></a>

## typography

| class                                           | description                                       |
| --------------------------------------------- | ------------------------------------------------- |
| `.fs-[01–10]`                                 | font-size from `.1em` to `10em` in `.1em` steps   |
| `.fw-[100–900]`                               | font-weight in hundreds (100 = thin, 900 = black) |
| `.thin`                                       | `font-weight: 300`                                |
| `.bold`                                       | `font-weight: 700`                                |
| `.uppercase` / `.lowercase` / `.capitalize`   | text-transform                                    |
| `.italic` / `.not-italic`                     | font-style control                                |
| `.text-[left|center|right|justify|start|end]` | text alignment                                    |
| `.text-vertical`                              | vertical writing mode                             |
| `.lh-[1–3]`                                   | line height (1, 1.1, 1.15, 1.2, 1.25, … , 3)  |
| `.underline` / `.line-through` / `.no-underline` | text decoration                                |
| `.decoration-[dotted|dashed|wavy]`            | text-decoration-style                             |
| `.underline-offset-[1|2]`                     | text-underline-offset in `.1em` steps             |
| `.truncate`                                   | truncate with ellipsis (alias)                    |
| `.overflow-ellipsis`                          | truncate with ellipsis                            |
| `.line-clamp-[1–4]`                           | clamp to N lines with ellipsis                    |
| `.indent-[1–3]`                               | text-indent in em                                 |
| `.whitespace-[pre|pre-wrap|pre-line|normal|nowrap]` | white-space control                     |
| `.break-words` / `.break-all` / `.break-keep`     | word-breaking                                  |

<a name="appearance"></a>

## appearance

| class                                                       | description                                                        |
| ----------------------------------------------------------- | ------------------------------------------------------------------ |
| `.bdr-[0–2]`                                                | border-radius in `.1em` steps                                      |
| `.bdr-circle`                                              | `border-radius: 50%`                                               |
| `.border-[s|m|l]`                                          | all borders                                                        |
| `.border-[top|bottom|left|right|x|y]-[0|s|m|l]`           | directional borders                                                |
| `.bg-[color]`                                              | background (`container`, `accent`, `red`, `blue`, `green`, …)      |
| `.text-[color]`                                            | text color (same values)                                           |
| `.shadow-[s|m|l]`                                         | box shadow                                                         |
| `.opacity-[0–1]`                                           | opacity (0, .1, .2 … 1)                                            |
| `.invert`                                                  | `filter: invert(1)`                                                |
| `.grayscale` / `.grayscale-0`                             | full / remove grayscale filter                                     |
| `.blur-[sm|md|lg|xl]`                                     | `filter: blur()`                                                   |
| `.brightness-[50|75|100|125|150]`                          | brightness filter                                                  |
| `.contrast-[50|75|100|125|150]`                            | contrast filter                                                    |
| `.saturate-[0|50|100|150|200]`                             | saturation filter                                                  |
| `.sepia`                                                    | sepia filter                                                       |
| `.hue-rotate-[15|30|60|90|180]`                           | hue-rotate filter                                                  |
| `.filter-none`                                              | remove all filters                                                 |
| `.backdrop-blur-[sm|md|lg|xl]`                            | backdrop blur filter                                               |
| `.object-[cover|contain|fill|none]`                       | image/video fitting                                                |
| `.object-[center|top|bottom|left|right]`                  | image/video position                                               |
| `.visible` / `.invisible`                                  | visibility (preserves layout space)                                |
| `.list-none` / `.list-disc` / `.list-decimal`            | list style type                                                    |
| `.list-inside` / `.list-outside`                            | list style position                                                |
| `.divide-y` / `.divide-x`                                   | border between children                                            |
| `.table`                                                   | styled table (header, rows, hover)                                 |
| `.table-bordered`                                          | `.table` variant with borders                                     |
| `.table-striped`                                           | `.table` variant with alternating rows                            |
| `.table-compact`                                           | `.table` variant with compact padding                             |
| `.table-auto` / `.table-fixed`                             | table layout mode                                                  |
| `.border-collapse` / `.border-separate`                    | border collapse mode                                               |
| `.border-spacing-0`                                        | remove border-spacing                                              |
| `.table-caption`                                          | table caption styling                                              |
| `.outline-none`                                            | `outline: none`                                                    |

<a name="interaction"></a>

## interaction

| class                                         | description                   |
| --------------------------------------------- | ----------------------------- |
| `.pointer`                                    | `cursor: pointer`             |
| `.no-events`                                  | `pointer-events: none`        |
| `.click-haptic-[small|med]`                   | scale-down effect on click    |
| `.hover:bg-[color]`                           | background change on hover    |
| `.hover:text-[color]`                         | text color change on hover    |
| `.shadow-hover-[s,m,l]`                       | shadow change on hover        |
| `.select-none` / `.select-text` / `.select-all` | user-select control        |
| `.appearance-none`                            | remove native form styling    |
| `.sr-only`                                    | visually hidden but accessible|
| `.float-left` / `.float-right` / `.float-none` | float                   |
| `.clear-both` / `.clear-left` / `.clear-right` / `.clear-none` | clear |
| `.align-top` / `.align-middle` / `.align-bottom` | vertical-align |
| `.align-text-top` / `.align-text-bottom` | vertical-align (text-relative) |
| `.align-sub` / `.align-super`                 | vertical-align for sub/superscript |

<a name="transitions--animation"></a>

## transitions & animation

| class                             | description                            |
| --------------------------------- | -------------------------------------- |
| `.tr-[01–1]`                      | transition duration from `.1s` to `1s` |
| `.fade-[in,out]`                  | opacity transition                     |
| `.slide-in-[left,right,up,down]`  | entry movement                         |
| `.slide-out-[left,right,up,down]` | exit movement                          |
| `.preload`                        | disables all transitions               |

<a name="performance"></a>

## performance

these classes communicate rendering hints to the browser. apply them to elements with complex, frequently updated content.

| class             | description                                                 |
| ----------------- | ----------------------------------------------------------- |
| `.stable`         | `will-change: transform; contain: paint`                    |
| `.self-contained` | `contain: layout paint`                                     |
| `.isolated`       | `contain: strict`                                           |
| `.gpu`            | `transform: translateZ(0); will-change: transform, opacity` |

<a name="dom-functions"></a>

## dom functions

global functions available in all page scripts and components. these should be preferred over native browser apis. they return cleanup functions and integrate with `dComponent`'s auto cleanup system.

### selection

```javascript
select('.btn-primary')              // querySelector shorthand
selectAll('.card')                  // querySelectorAll, returns a true array
select('.input', formElement)       // scoped to a parent element
```

### events

```javascript
const cleanup = listen(button, 'click', handler);
cleanup(); // detach manually if needed

listenAll(selectAll('.btn'), 'click', handler);
```

### classes

```javascript
addClass('active', el)
removeClass('hidden', el)
toggleClass('open', el, state) // state is optional
hasClass('disabled', el)  // returns boolean
```

### timing

```javascript
await nextFrame(); // resolves after 2 raf cycles (safe for measuring after dom changes)
await sleep(300);  // promise based delay
```

<a name="data-functions"></a>

## data functions

### escaping

```javascript
escapeHtml(v)               // escapes html entities (&, <, >, ", ')
escapeJs(v)                 // escapes quotes, backticks, backslashes, dollar signs, control chars, and </script> tags
```

### type checking

```javascript
isString(v)    isNumber(v)    isBoolean(v)
isArray(v)     isObject(v)    isFunction(v)
isEmpty(v)     // true for "", [], {}, null, undefined
hasValue(v)    // true if not empty, null, or undefined
```

### array utilities

```javascript
move(input, from, to)       // move an item within an array or string
remove(input, index)        // remove an item at index
replace(input, index, val)  // replace an item at index
limit(input, max)           // slice to a maximum length
uniquify(input)             // remove duplicates from an array or string

asyncForEach(arr, fn)       // serial async iteration (awaits each fn before continuing)
```

### extras

```javascript
random(1, 100)              // inclusive integer random
uniqueId(8)                 // collision resistant unique id string
clipboard('text to copy')   // write to system clipboard
formatTime(seconds)         // returns "mm:ss"
formatDate(date, false)     // returns "YYYY-MM-DD"
formatDate(date)            // returns "YYYY-MM-DD HH:mm:ss"
localizeDate(date, false)   // returns "YYYY-MM-DD" in the user's timezone
localizeDate(date)          // returns "YYYY-MM-DD HH:mm:ss" in the user's timezone
distance(a, b)              // levenshtein or jaccard distance between two strings

Storage.get('key')          // typed localStorage wrapper
Storage.set('key', value)
Storage.remove('key')
Storage.clear()
```

<a name="notify--modal"></a>

## notify & modal

### notify

displays a non blocking toast notification. auto dismisses after the timer expires.

```javascript
notify('profile updated');
notify('upload failed — please try again', 6000); // custom duration in ms
```

pass a setup callback to attach interaction to notification elements. use `.listen()` inside the callback for automatic cleanup when the notification closes:

```javascript
notify('<button class="undo-btn">undo</button>', 5000, (n) => {
  n.listen(n.querySelector('.undo-btn'), 'click', () => {
    undoLastAction();
    n.close();
  });
});

// omit the timer to control close timing from the callback
notify('<button class="undo-btn">undo</button>', (n) => {
  n.listen(n.querySelector('.undo-btn'), 'click', () => n.close());
});
```

### modal

displays a floating modal. handles its opening animation and removes itself from the dom after closing.

```javascript
modal('<h2>success</h2><p>your order has been placed.</p>');
```

pass a setup callback for interactive modals. use `.listen()` for automatic cleanup:

```javascript
modal(`
  <h2>delete post</h2>
  <p>this action cannot be undone.</p>
  <button class="btn-danger confirm">delete</button>
  <button class="btn cancel">cancel</button>
`, (m) => {
  m.listen(m.querySelector('.confirm'), 'click', async () => {
    await fetch(`/posts/${postId}`, { method: 'DELETE' });
    m.close();
    notify('post deleted');
  });

  m.listen(m.querySelector('.cancel'), 'click', () => m.close());
});
```
