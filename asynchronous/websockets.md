# websockets

- [websockets](#websockets)
  - [introduction](#introduction)
  - [how it works](#how-it-works)
  - [defining socket events](#defining-socket-events)
    - [event handlers](#event-handlers)
    - [inline handlers](#inline-handlers)
    - [controller handlers](#controller-handlers)
  - [middleware](#middleware)
    - [inline middleware](#inline-middleware)
    - [grouped middleware](#grouped-middleware)
  - [responding to events](#responding-to-events)
    - [rendering html](#rendering-html)
    - [sending json](#sending-json)
    - [redirects](#redirects)
    - [aborting](#aborting)
    - [cookies](#cookies)
  - [target restrictions](#target-restrictions)
  - [broadcasting](#broadcasting)
  - [the socket facade](#the-socket-facade)
  - [client side usage](#client-side-usage)
    - [connecting](#connecting)
    - [emitting events](#emitting-events)
    - [listening for events](#listening-for-events)
  - [origin protection](#origin-protection)
  - [debug mode](#debug-mode)

<a name="introduction"></a>

## introduction

dframework ships with a fully integrated socket layer built on top of the `ws` library. it provides a dedicated event router that mirrors the http `Route` architecture, complete with middleware support, controller resolution, session authentication, and html rendering. the socket server boots automatically alongside your http server. there is nothing to install separately.

on the client side, the framework's frontend runtime exposes a global `Socket` facade that manages the connection, event emission, and event listening. the socket connection is established automatically when the spa router initializes.

for information on `d-wire` and `d-live` reactive elements that build on top of this socket layer, see the [reactivity](/documentation/reactivity) documentation.

<a name="how-it-works"></a>

## how it works

when your application starts, the framework attaches a socket upgrade handler to the http server on the `/ws` path. when a browser opens a websocket connection, the framework creates a full `Request` object from the upgrade request, giving you access to cookies, sessions, headers, and the authenticated user inside your socket event handlers.

incoming messages are expected as json objects with an `event` property. the socket router looks up the registered handler for that event name and runs it through the middleware pipeline, exactly as an http route would be processed.

<a name="defining-socket-events"></a>

## defining socket events

socket events are defined in your route files, typically `routes/wire.js`. you register events using the global `Socket` facade, which delegates to the underlying `SocketRouter` instance.

```javascript
import { Socket } from 'dframework';
```

<a name="event-handlers"></a>

### event handlers

the `Socket.on()` method registers a handler for a named event. the event name uses colon separated namespacing by convention.

```javascript
Socket.on('chat:message', (req) => {
  const { text } = req.body;
  return json({ received: true, text });
});
```

the handler receives the same arguments as an http route handler. `req` contains the full request object with access to `req.body` (the event payload), `req.user`, `req.session`, `req.cookies`, and all standard request methods. the global response helpers (`render()`, `json()`, `redirect()`, `abort()`, `cookie()`) work inside socket handlers exactly as they do in http controllers.

if the handler returns a value without calling any response method, the framework automatically wraps it in a json response with a 200 status code.

<a name="inline-handlers"></a>

### inline handlers

for simple events, you can use an inline function directly.

```javascript
Socket.on('ping', () => {
  return json({ pong: true, time: Date.now() });
});
```

<a name="controller-handlers"></a>

### controller handlers

for complex logic, reference a controller method using the familiar `Controller@method` string syntax. the framework resolves and imports the controller dynamically.

```javascript
Socket.on('chat:message', 'ChatController@onMessage');
Socket.on('chat:typing', 'ChatController@onTyping');
```

```javascript
export default class ChatController {
  async onMessage(req) {
    const { text } = req.body;
    await Message.create({ user_id: req.user.id, text });
    Socket.broadcast('chat:new', { text, user: req.user.name });
  }

  async onTyping() {
    return json({ typing: true });
  }
}
```

socket wire controllers are stored in the same `controllers` directory as http controllers and follow the same conventions. a single controller can handle both http routes and socket events.

<a name="middleware"></a>

## middleware

socket events support the same middleware system as http routes. middleware functions receive `req` and `next`, and must call `next()` to proceed to the handler.

<a name="inline-middleware"></a>

### inline middleware

you can attach middleware to a single event by passing an array where the last element is the handler and all preceding elements are middleware references.

```javascript
Socket.on('admin:action', ['AuthMiddleware@requireAuth', 'AdminController@execute']);
```

<a name="grouped-middleware"></a>

### grouped middleware

the `Socket.group()` method applies shared middleware to multiple events at once.

```javascript
Socket.group({ middleware: 'AuthMiddleware@requireAuth' }, (auth) => {
  auth.on('profile:update', 'ProfileController@update');
  auth.on('profile:avatar', 'ProfileController@avatar');
});
```

you can also chain middleware using the `Socket.middleware()` method for a more fluent api.

```javascript
Socket.middleware('AuthMiddleware@requireAuth').group({}, (auth) => {
  auth.on('dashboard:refresh', 'DashboardController@refresh');
});
```

here is how socket events look in a real application:

```javascript
import { Socket } from 'dframework';

Socket.on('pair:status', 'auth.AuthController@wire_status').targets(['#qr-box']);

Socket.group({ middleware: ['AuthMiddleware@requireAuth'] }, (auth) => {
  auth.on('follow:toggle', 'app.FriendsController@toggle').targets(['#leaderboard-list', '#friends-list-container']);
  auth.on('friends:tab', 'app.FriendsController@friends_tab').targets(['#friends-list-container']);
  auth.on('playback:heartbeat', 'app.PlaybackController@handleHeartbeat');
});
```

<a name="responding-to-events"></a>

## responding to events

socket handlers use the same global response helpers available in http controllers. all responses are delivered over the socket connection.

<a name="rendering-html"></a>

### rendering html

the `render()` helper compiles a view template and sends the resulting html to the client. the framework morphs the html into the specified target element on the client side without a page refresh.

```javascript
Socket.on('feed:refresh', async () => {
  const posts = await Post.latest(20);
  return render('partials.feed', { posts });
});
```

the rendered html is sent as a `d-wire:render` event. the client side runtime parses it and morphs the target dom element, preserving focus state, input values, and scroll positions.

the framework uses payload hashing to prevent redundant dom updates. if the rendered html produces the same hash as the previous render for that target, the update is silently skipped.

<a name="sending-json"></a>

### sending json

the `json()` helper sends a json payload back to the client.

```javascript
Socket.on('stats:fetch', async () => {
  const count = await User.count();
  return json({ users: count });
});
```

<a name="redirects"></a>

### redirects

the `redirect()` helper sends a redirect instruction to the client. the browser will navigate to the specified url.

```javascript
Socket.on('session:expired', () => {
  return redirect('/login');
});
```

<a name="aborting"></a>

### aborting

the `abort()` helper sends an error response with a status code and optional message.

```javascript
Socket.on('admin:action', (req) => {
  if (!req.user?.isAdmin) {
    return abort(403, 'forbidden');
  }
});
```

the default status messages are `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`, and `500 Internal Error`.

<a name="cookies"></a>

### cookies

you can set cookies from a socket handler using the `cookie()` helper. the cookies are sent alongside the response and the client side runtime applies them to the browser.

```javascript
Socket.on('theme:set', (req) => {
  cookie('theme', req.body.theme, { path: '/', sameSite: 'Lax' });
  return json({ ok: true });
});
```

<a name="target-restrictions"></a>

## target restrictions

you can restrict which dom target selectors an event is allowed to render into using the `targets()` chain method. if the client sends a target that is not in the allowed list, the event is silently rejected.

```javascript
Socket.on('sidebar:update', 'SidebarController@render').targets('#sidebar');
Socket.on('panel:refresh', 'PanelController@render').targets(['#left-panel', '#right-panel']);
```

<a name="broadcasting"></a>

## broadcasting

the `Socket.broadcast()` method sends an event to all connected websocket clients. you can optionally exclude a specific connection (typically the sender).

```javascript
Socket.on('chat:send', (req, res, ws) => {
  const message = { text: req.body.text, user: req.user.name };
  Socket.broadcast('chat:new', message, ws);
  return json({ sent: true });
});
```

the third argument is the websocket connection to exclude from the broadcast. passing the current `ws` prevents the sender from receiving their own message.

<a name="the-socket-facade"></a>

## the socket facade

the `Socket` global is a proxy facade that delegates all method calls to the application's `SocketRouter` instance. it is available globally after the application initializes and can be imported from the framework.

```javascript
import { Socket } from 'dframework';
```

the facade exposes all `SocketRouter` methods: `on()`, `group()`, `middleware()`, `broadcast()`, and `liveNotify()`.

<a name="client-side-usage"></a>

## client side usage

the framework's frontend runtime provides a global `Socket` object on the `window` that manages the websocket connection.

<a name="connecting"></a>

### connecting

the websocket connection is established automatically when the spa router initializes. you can also connect manually.

```javascript
await Socket.connect();
```

the connection url defaults to `/ws` and automatically uses `wss:` on secure pages and `ws:` on insecure ones. the full uri is constructed from the current page's `location.host`.

<a name="emitting-events"></a>

### emitting events

use `Socket.emit()` to send an event to the server. you can pass a data payload and an optional target selector.

```javascript
Socket.emit('chat:send', { text: 'hello world' });
```

```javascript
Socket.emit('dashboard:refresh', { filter: 'today' }, '#dashboard');
```

when a target is specified, the server handler's rendered output will be morphed into that specific dom element.

<a name="listening-for-events"></a>

### listening for events

use `Socket.on()` on the client side to listen for events pushed from the server.

```javascript
Socket.on('chat:new', (data) => {
  appendMessage(data.text, data.user);
});
```

for broadcast events, the callback receives the data payload directly.

<a name="origin-protection"></a>

## origin protection

the socket server validates the `Origin` header of every upgrade request against your configured `APP_URL`. connections from mismatched origins are immediately destroyed to prevent cross-site websocket hijacking (cswsh) attacks.

in the `local` environment, connections from `localhost` and `127.0.0.1` are allowed regardless of the configured url.

<a name="debug-mode"></a>

## debug mode

in the `local` environment with debug mode enabled, the socket router attaches detailed timing and query information to every response. the debugbar on the client side displays the elapsed time, executed queries, and request headers for each socket event, providing the same level of visibility as http requests.