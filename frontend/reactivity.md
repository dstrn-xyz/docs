# reactivity

- [introduction](#introduction)

- [choosing a system](#choosing-a-system)

- [d-wire event driven reactivity](#d-wire-event-driven-reactivity)

  - [passing data](#passing-data)
  - [backend handler](#backend-handler)
  - [trigger types](#trigger-types)
  - [programmatic dispatch](#programmatic-dispatch)

- [d-live state driven reactivity](#d-live-state-driven-reactivity)

  - [filtering](#filtering)
  - [how it works under the hood](#how-it-works-under-the-hood)

- [surgical state sync](#surgical-state-sync)

- [error handling](#error-handling)

<a name="introduction"></a>

## introduction

dframework provides two complementary reactivity systems that keep the user interface in sync with server state over the socket layer. no client side rendering framework is involved. the server renders html and the framework delivers it surgically to the correct dom target.

<a name="choosing-a-system"></a>

## choosing a system

| feature   | d-wire                          | d-live                                |
| --------- | ------------------------------- | ------------------------------------- |
| trigger   | an explicit user action         | a database write                      |
| best for  | counters, search, modals, forms | feeds, dashboards, live lists         |
| requires  | a named socket handler          | a d-live attribute, no handler        |
| rendering | the handler calls render        | the framework refetches automatically |

use d-wire when the user expects a specific result from a deliberate action. use d-live when a region must stay in sync with data regardless of what caused it to change.

<a name="d-wire-event-driven-reactivity"></a>

## d-wire event driven reactivity

attach `d-wire` to any html element. when triggered, the element emits a named event over the socket to a socket handler. the handler renders a partial and the framework replaces the target container with the result.

```html
<button d-wire="counter:increment" d-target="#counter">+1</button>
```

<a name="passing-data"></a>

### passing data

use `d-wire-data` to send a payload with the event. it accepts key value pairs separated by commas or raw json.

```html
<button
  d-wire="post:like"
  d-wire-data="id:{{ post.id }}, type:'post'"
  d-target="#likes-{{ post.id }}">
  like
</button>

<button
  d-wire="user:update"
  d-wire-data='{"id": {{ user.id }}, "status": "active"}'
  d-target="#user-status">
  activate
</button>
```

the payload is merged with any form data present and accessible as the request body in the handler.

<a name="backend-handler"></a>

### backend handler

a `d-wire` handler must call render or json. the rendered html replaces the target element on the client. the framework hashes the rendered output and suppresses the update if the content has not changed, preventing redundant dom operations.

to prevent arbitrary dom injection, you must restrict which html elements a route can update by chaining `targets()` to your route definition.

```javascript
Socket.on('counter:increment', async (req, res) => {
  const count = await Counter.increment(req.body.id);
  return res.render('partials.counter', { count });
}).targets(['#counter']);

Socket.on('user:search', async (req, res) => {
  const users = await User.where('name', 'LIKE', `%${req.body.q}%`).limit(10).get();
  return res.render('partials.user-list', { users });
}).targets(['#user-list']);
```

<a name="trigger-types"></a>

### trigger types

`d-wire` binds to the natural event for each element type.

| element               | default trigger |
| --------------------- | --------------- |
| `<button>`, `<a>`     | `click`         |
| `<form>`              | `submit`        |
| `<input>`, `<select>` | `change`        |

override with `d-trigger`.

```html
<!-- fire on keyup with debounce handled server side -->
<input d-wire="user:search" d-target="#results" d-trigger="keyup" placeholder="search users">
```

special trigger values are available.

| value       | behaviour                                                 |
| ----------- | --------------------------------------------------------- |
| `load`      | fires once immediately when the element mounts            |
| `poll-{ms}` | fires continuously every n milliseconds, e.g. `poll-5000` |

<a name="programmatic-dispatch"></a>

### programmatic dispatch

emit a `d-wire` event from javascript or from inside a frontend component.

```javascript
Socket.emit('dashboard:refresh', {}, '#stats-panel');

// from a component
this.wire('notification:dismiss', { id: this.notificationId });
```

<a name="d-live-state-driven-reactivity"></a>

## d-live state driven reactivity

`d-live` subscribes a dom element to a database table. whenever any model write touches that table, the framework evaluates active subscriptions, notifies matching clients, and the element refetches and replaces its own content. no backend handler is needed.

```html
<div d-live="posts">
  @foreach(const post of await Post.where({ status: 'published' }).orderBy('created_at', 'DESC').get())
    <article>
      <h2>{{ post.title }}</h2>
      <p>{{ post.excerpt }}</p>
    </article>
  @endforeach
</div>
```

multiple elements can subscribe to the same or different tables simultaneously on a single page.

<a name="filtering"></a>

### filtering

`d-live-filter` restricts updates to rows matching specific criteria. the server evaluates filters against the changed row before notifying clients. clients that do not match the filter receive no message and perform no refetch.

```html
<div d-live="orders" d-live-filter="user_id:{{ user.id }}">...</div>
```

supported operators include.

| operator      | example             | meaning               |
| ------------- | ------------------- | --------------------- |
| `=` (default) | `status:published`  | equals                |
| `!=`          | `status:!=archived` | not equal             |
| `>`           | `price:>100`        | greater than          |
| `<`           | `count:<5`          | less than             |
| `>=`          | `rating:>=4`        | greater than or equal |
| `<=`          | `stock:<=10`        | less than or equal    |

multiple filters use and logic. use commas to separate them.

```html
<div d-live="messages" d-live-filter="room_id:{{ room.id }},status:visible">...</div>
```

<a name="how-it-works-under-the-hood"></a>

### how it works

1. on mount, the element sends a subscribe message over the socket, registering itself for the table with any filter criteria.
2. when the query builder executes a write, it notifies the socket router with the table name and the changed data.
3. the router iterates active subscriptions, evaluates each filter against the changed data, and emits a notification only to clients whose filter matched.
4. the subscribed element rerenders its own content. updates are debounced to one hundred milliseconds to handle bulk operations efficiently. updates are suppressed for two hundred and fifty milliseconds after a successful form submission to prevent double fetches.

<a name="surgical-state-sync"></a>

## surgical state sync

push state directly into a component's reactive memory without rerendering its template. this is useful for lightweight state updates that do not require a full html replacement.

```javascript
// backend, emit to a specific component by selector
Socket.emit('d-sync:state', { unread: 14 }, '#notification-badge');
```

the targeted component calls its set state method automatically, triggering its render and effect loops.

<a name="error-handling"></a>

## error handling

spa and reactivity integrate with the abort helper.

form submissions use standard fetch requests. calling the abort method returns a structured json error object that the form renders inline next to the relevant inputs.

spa navigations use background requests. calling the abort method renders the full error page html, which the spa layer displays and reflects in the url. socket routing handlers also gracefully catch and log any rendering exceptions, falling back to a client safe error message when necessary.