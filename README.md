<p align="center">
  <a href="https://framework.dstrn.xyz" target="_blank"><img src="https://raw.githubusercontent.com/dstrn-xyz/docs/refs/heads/main/.images/text_logo.png" width="400"></a>
</p>
<p align="center">
a fully integrated javascript application framework built around elegance by default.
</p>

<p align="center">
  <a href="https://github.com/dstrn825/dframework2/actions"><img src="https://github.com/dstrn825/dframework2/actions/workflows/node.js.yml/badge.svg" alt="build status"></a>
</p>

## about dframework

dframework is an opinionated framework that enforces architectural consistency and deterministic behavior. it controls the complete application lifecycle including routing, rendering, components, styling, assets, state management, compilation, transport, and native runtime integration.

plugin ecosystems, adapter layers, and competing architectural patterns are intentionally rejected. dframework is designed to be used as a complete framework without partial adoption.

<a name="who-is-dframework-for"></a>

# who dframework is for

dframework targets developers who value integrated tooling, enforced conventions, architectural consistency, and minimal configuration. it provides deterministic application structure, long term maintainability, and framework controlled optimizations.

it may not suit developers who prefer custom stacks, plugin ecosystems, interchangeable tooling, progressive adoption, or extensive runtime customization.

<a name="developing-with-dframework"></a>

# developing with dframework

dframework removes architectural decision making, allowing developers to focus on application logic rather than tooling assembly.

routes are defined declaratively:

```javascript
Route.get('/', 'HomeController@index');
Route.post('/users', 'UserController@store').name('users.store');
```

controllers handle request logic:

```javascript
export default class UserController extends Controller {
  async store(req) {
    const validation = validate(req.body, {
      email: 'required|email',
      password: 'required|min:8'
    });

    if (validation.fails()) {
      return abort(422).withErrors(validation.errors());
    }

    const user = await User.create({
      email: req.body.email,
      password: req.body.password
    });

    return json({ user });
  }
}
```

views use a compiled templating system:

```html
@extends('layouts.app')

@section('content')
  <div d-live="users" d-live-filter="active:1">
    @foreach(await User.where({ active: 1 }) as user)
      <div class="flex-row gap-05">
        {{ user.email }}
        <d-hold-button
          d-wire="user:destroy"
          d-wire-data="id:{{ user.id }}">delete</d-hold-button>
      </div>
    @endforeach
  </div>
@endsection
```

native application workflows:

```bash
dstrn simulate --ios
dstrn build --ios
```

<a name="quick-start"></a>

# quick start

```bash
npm install -g dframework
dstrn init my-app
cd my-app
dstrn serve
```

<a name="philosophy"></a>

# philosophy

dframework follows core principles that shape its architecture:

## architectural consistency

applications should not require extensive architectural decisions. directory structure, lifecycle behavior, rendering flow, and framework conventions are intentionally enforced to ensure structural consistency across all dframework applications.

## closed world architecture

dframework assumes complete ownership of the application runtime. routing, rendering, styling, state management, compilation, and transport form a single coherent system without support for partial adoption.

## performance through coherence

performance emerges from architectural coherence and assumption density. by controlling the full lifecycle, the framework aggressively optimizes routing, middleware dispatch, rendering, serialization, hydration, template compilation, and memory allocation as a unified runtime rather than a collection of interchangeable libraries.

## convention over configuration

configuration is intentionally minimal. directory structure, lifecycle patterns, and architectural conventions are enforced by the framework, with configurations provided only when they reinforce this philosophy.

## forward evolution

legacy compatibility layers are intentionally avoided. when architectural improvements require breaking changes, applications are expected to migrate forward, prioritizing internal consistency over long term backward compatibility.

<a name="non-goals"></a>

# non goals

the framework intentionally excludes:

- react, vue, angular
- bootstrap, tailwind
- jquery, axios
- vite and external bundler pipelines
- plugin systems and adapter layers
- alternate rendering engines and routing systems
- configurable directory structures
- partial framework adoption

these restrictions are fundamental to maintaining architectural integrity.

<a name="why-dframework"></a>

# why dframework

modern development increasingly relies on fragmented tooling, layered abstractions, and interchangeable architectures, leading to configuration fatigue, dependency instability, architectural inconsistency, runtime overhead, and unpredictable application structure.

dframework takes an opposing approach by prioritizing coherence, predictability, deterministic architecture, integrated tooling, runtime efficiency, and long term maintainability.

constraints in dframework exist to strengthen system integrity rather than limit developers arbitrarily.

<a name="features"></a>

# features

dframework provides:
- integrated server side rendering
- surgical spa navigation
- reactivity systems
- native websocket support
- orm and query builder
- view compilation and templating
- asset compilation and minification
- utility styling system
- high performance UI components
- background jobs and scheduling
- localization and translations
- sessions, csrf, and security tooling
- native desktop and mobile runtime integration

<a name="performance"></a>

# performance

dframework achieves performance through its closed world runtime architecture with strong assumptions. by controlling the complete lifecycle, the framework optimizes across subsystem boundaries rather than relying on interchangeable abstractions.

benchmark results and methodology:

- [benchmarks](/benchmarks/benchmarks.md)

<a name="documentation"></a>

# documentation

## fundamentals

- [getting started](/getting-started/getting-started.md)
- [project structure](/getting-started/project-structure.md)
- [routing](/fundamentals/routing.md)
- [controllers](/fundamentals/controllers.md)
- [middleware](/fundamentals/middlewares.md)
- [request lifecycle](/fundamentals/request-lifecycle.md)
- [views](/fundamentals/views.md)

## data management

- [database & models](/database/database.md)
- [migrations](/database/migrations.md)
- [query builder](/database/query-builder.md)
- [storage](/database/storage.md)

## frontend development

- [spa router](/frontend/spa-router.md)
- [websockets](/asynchronous/websockets.md)
- [components](/frontend/frontend-components.md)
- [reactivity](/frontend/reactivity.md)
- [frontend utilities](/frontend/frontend-utilities.md)
- [frontend pipeline](/frontend/frontend-pipeline.md)

## asynchronous processing

- [jobs and queues](/asynchronous/jobs-and-queues.md)
- [scheduling and commands](/asynchronous/scheduling-and-commands.md)

## tooling

- [cli](/other/cli.md)
- [localization](/other/localization.md)
- [logging and errors](/other/logging-and-errors.md)

## security

- [security](/security/security.md)
- [authentication](/security/authentication.md)
- [sessions](/security/sessions.md)
- [validation](/security/validation.md)

## native applications

- [getting started](/going-native/getting-started.md)
- [bridge plugins](/going-native/bridge-plugins.md)
- [build and distribution](/going-native/build-distribution.md)
- [simulation and debugging](/going-native/simulation-debugging.md)

<a name="contributing"></a>

# contributing

before contributing, understand that dframework is intentionally opinionated.

features that weaken architectural consistency, introduce alternate paradigms, or reduce framework control over the runtime will be rejected.

all contributions must align with the framework philosophy.

<a name="license"></a>

# license

[mit](/license.md)
