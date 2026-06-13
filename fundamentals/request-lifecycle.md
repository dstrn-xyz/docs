# request lifecycle

- [introduction](#introduction)

- [lifecycle overview](#lifecycle-overview)

  - [the fast path](#the-fast-path)
  - [the request pipeline](#the-request-pipeline)
  - [static files and assets](#static-files-and-assets)
  - [routing and middleware](#routing-and-middleware)
  - [dispatching to controllers](#dispatching-to-controllers)

<a name="introduction"></a>

## introduction

understanding how requests flow through dframework is critical for building predictable applications. the framework operates as a closed loop. every incoming http request is intercepted parsed and routed internally by the framework engine. you do not need to configure external web servers for basic request handling during development.

<a name="lifecycle-overview"></a>

## lifecycle overview

when a request enters the application it first hits the central application handler. the framework immediately determines if the application is running in a production environment with caching enabled.

<a name="the-fast-path"></a>

### the fast path

if the fast path is active the request bypasses several initialization steps and jumps directly into the pre compiled routing tables stored in compiled `.rc` files. this aggressive optimization drastically reduces latency for static routes and cached responses. the framework generates these highly optimized files during the production build step.

```javascript
// internal framework code demonstrating the fast path execution
if (matchedStatic && matchedStatic._compiled) {
  request.params = {};
  
  // instantly execute the compiled .rc handler
  const result = RequestContext.run(request, () =>
    matchedStatic._compiled(request, response, this, this.router)
  );
  return this._routeResult(result, req, res);
}
```

<a name="the-request-pipeline"></a>

### the request pipeline

if the fast path is not applicable the request enters the request pipeline. the pipeline is a specialized component that evaluates the request against internal stages before it ever reaches your application logic.
it first checks if the request is asking for internal assets like the debug bar or the shield security scripts. if a match is found the pipeline serves the asset directly from memory.

<a name="static-files-and-assets"></a>

### static files and assets

next the pipeline evaluates static files. if static file serving is enabled the pipeline checks the `public` directory. if a valid file exists it streams the file back to the client with the appropriate caching headers. if the request does not match any static files the pipeline hands control over to the application router.

> \[!NOTE]
> during local development the internal server acts as a fully capable static file server. in production it is recommended to let nginx or a cdn handle static assets.

<a name="routing-and-middleware"></a>

### routing and middleware

the router is the core decision making engine. it receives the raw request wraps it in a managed request context and matches the url against your defined routes.

once a route is matched the router begins processing middleware. middleware forms an execution chain that the request must pass through. if any middleware aborts or returns a response the chain halts immediately and the response is sent back to the client.

<a name="dispatching-to-controllers"></a>

### dispatching to controllers

finally if all middleware passes the request reaches your designated controller. controllers in the framework are simple classes defining async methods. arguments like the request and response are optional making your business logic extremely clean. the result is then transformed into a managed response object which is ultimately flushed back to the client completing the lifecycle.

```javascript
// controllers/UserController.js
export default class UserController {
  async show(req) {
    const user = await User.find(req.params.id);
    
    // the json response is flushed directly back to the client
    return json({ user });
  }
}
```
