# d-virtual-list

dom virtualization component skipping layout and paint for offscreen elements using css content visibility and automatic bottom threshold detection.

```html
<d-virtual-list item-size="64px" buffer="800px 0px 800px 0px" threshold="600">
  @foreach(const item of items)
    <div class="p-3 bg-container mb-2 rounded">{{ item.title }}</div>
  @endforeach
</d-virtual-list>
```

<a name="attributes-and-properties-d-virtual-list"></a>
## attributes and properties

| attribute       | property       | type     | default                 | form | description                                                           |
| :-------------- | :------------- | :------- | :---------------------- | :--- | :-------------------------------------------------------------------- |
| `item-size`     | `itemSize`     | `string` | `"52px"`                | no   | estimated item height fallback for contain intrinsic size             |
| `buffer`        | `buffer`       | `string` | `"600px 0px 600px 0px"` | no   | intersection observer root margin for pre rendering offscreen items   |
| `threshold`     | `threshold`    | `number` | `500`                   | no   | pixel distance to bottom required to fire reachbottom                 |
| `cooldown`      | `cooldown`     | `number` | `250`                   | no   | minimum delay in milliseconds between reachbottom event dispatches    |
| `scroll-parent` | `scrollParent` | `string` | `""`                    | no   | optional css selector targeting custom scroll container               |

<a name="programmatic-api-d-virtual-list"></a>
## programmatic api

```javascript
const list = select('d-virtual-list');

list.itemSize = '72px';
list.threshold = 400;

// manually trigger bottom proximity calculation
list.checkBottom();
```

<a name="events-d-virtual-list"></a>
## events

| event         | detail                    | bubbles | description                                                    |
| :------------ | :------------------------ | :------ | :------------------------------------------------------------- |
| `reachbottom` | `{ distance, threshold }` | yes     | dispatched when the user scrolls near the bottom of the list  |

<a name="behavior-d-virtual-list"></a>
## behavior

- virtualizes offscreen items by applying `content-visibility: hidden` and `contain-intrinsic-size` while keeping light dom nodes intact.
- detects scroll container via parent hierarchy, `scroll-parent` selector, or window viewport.
- uses an anchor element and intersection observer to reliably detect bottom proximity and emit `reachbottom` events for infinite scrolling.
- clears and pauses bottom checks when offscreen.

---

<a name="overriding-built-in-components"></a>
