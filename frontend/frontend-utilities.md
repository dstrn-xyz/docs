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
| `.z-[0–auto]`            | z-index: `0`, `1`, `2`, `3`, `4`, `5`, `10`, `50`, `100`, `auto` |

### grid

| class          | description          |
| -------------- | -------------------- |
| `.grid-center` | centered grid        |
| `.grid-2`      | 2-column grid        |
| `.grid-3`      | 3-column grid        |
| `.grid-4`      | 4-column grid        |
| `.grid-auto`   | dense auto-flow grid |

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
| `.justify-[start,end,center,between,around,evenly]` | main axis alignment           |
| `.align-[start,end,center,baseline,stretch]`        | cross axis alignment          |
| `.self-[start,end,center,stretch,baseline]`         | per item cross axis alignment |

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
| `.g-`           | gap (no `auto`)      |

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

| class                                         | description                                     |
| --------------------------------------------- | ----------------------------------------------- |
| `.fs-[01–10]`                                 | font size from `.1em` to `10em` in `.1em` steps |
| `.thin`                                       | `font-weight: 300`                              |
| `.bold`                                       | `font-weight: 700`                              |
| `.uppercase`                                  | `text-transform: uppercase`                     |
| `.lowercase`                                  | `text-transform: lowercase`                     |
| `.capitalize`                                 | `text-transform: capitalize`                    |
| `.text-[left,center,right,justify,start,end]` | text alignment                                  |
| `.text-vertical`                              | vertical writing mode                           |
| `.overflow-ellipsis`                          | truncate with ellipsis                          |
| `.line-clamp-[1–4]`                           | clamp to N lines with ellipsis                  |

<a name="appearance"></a>

## appearance

| class                                           | description                                                   |
| ----------------------------------------------- | ------------------------------------------------------------- |
| `.bdr-[0–2]`                                    | border radius in `.1em` steps                                 |
| `.bdr-circle`                                   | `border-radius: 50%`                                          |
| `.border-[s,m,l]`                               | all borders                                                   |
| `.border-[top,bottom,left,right,x,y]-[0,s,m,l]` | directional borders                                           |
| `.bg-[color]`                                   | background (`container`, `accent`, `red`, `blue`, `green`, …) |
| `.text-[color]`                                 | text color (same values)                                      |
| `.shadow-[s,m,l]`                               | box shadow                                                    |
| `.opacity-[0–1]`                                | opacity levels                                                |
| `.invert`                                       | `filter: invert(1)`                                           |

<a name="interaction"></a>

## interaction

| class                       | description                |
| --------------------------- | -------------------------- |
| `.pointer`                  | `cursor: pointer`          |
| `.no-events`                | `pointer-events: none`     |
| `.click-haptic-[small,med]` | scale-down effect on click |
| `.hover:bg-[color]`         | background change on hover |
| `.hover:text-[color]`       | text color change on hover |
| `.shadow-hover-[s,m,l]`     | shadow change on hover     |

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
addClass(el, 'active')
removeClass(el, 'hidden')
toggleClass(el, 'open')
hasClass(el, 'disabled')  // returns boolean
```

### timing

```javascript
await nextFrame(); // resolves after 2 raf cycles (safe for measuring after dom changes)
await sleep(300);  // promise based delay
```

<a name="data-functions"></a>

## data functions

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
