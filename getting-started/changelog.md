# changelog

- [v0.21.0](#v0-21-0)
  - [core runtime and transport](#core-runtime-and-transport)
  - [declarative routing and middleware](#declarative-routing-and-middleware)
  - [database and active record](#database-and-active-record)
  - [view engine and compiler](#view-engine-and-compiler)
  - [frontend pipeline and dspa router](#frontend-pipeline-and-dspa-router)
  - [security and sessions](#security-and-sessions)
  - [queues and task scheduler](#queues-and-task-scheduler)
  - [native bridge and platform runtimes](#native-bridge-and-platform-runtimes)
  - [cli and architecture analyzer](#cli-and-architecture-analyzer)
- [historical development (2019 to 2026)](#historical-development)
  - [development milestones (v0.1.0 to v0.20.0)](#development-milestones)

<a name="v0-21-0"></a>

## v0.21.0

v0.21.0 marks the consolidated production release of dframework, bringing together the complete closed world architecture, native transport addon, compiled templating engine, dspa client router, and cross platform native runtimes.

<a name="core-runtime-and-transport"></a>

### core runtime and transport

- high performance custom native transport addon (`dfws`) compiled with libuv event loop integration and stripped tcp optimizations
- automatic, silent fallback to node built in http server when native binaries are unavailable on target platforms
- centralized application boot lifecycle managing configuration detection, logging, router initialization, and request dispatch
- explicit response helpers (`return json()`, `return render()`, `return redirect()`, `return back()`, `return abort()`, `return status()`) attached to global scope
- request context abstraction supporting headers, cookies, query parameters, multipart file uploads, and json payloads

<a name="declarative-routing-and-middleware"></a>

### declarative routing and middleware

- fluent declarative routing facade (`Route.get()`, `Route.post()`, `Route.put()`, `Route.delete()`, `Route.patch()`, `Route.resource()`)
- route group scoping supporting path prefixes, named route prefixes, and shared middleware pipelines
- compiled regular expression route matching engine for low overhead route dispatch
- pipeline based middleware runner executing pre route and post route handlers
- centralized error handling and custom error template rendering (`404.d`, `500.d`, `debug.d`)

<a name="database-and-active-record"></a>

### database and active record

- mysql2 connection pooling with automatic reconnection and async query execution
- active record model base class with fluent querying (`where()`, `find()`, `create()`, `update()`, `delete()`, `first()`, `paginate()`)
- database relationships supporting `hasOne`, `hasMany`, `belongsTo`, and `belongsToMany`
- fluent schema definition and migration engine (`dstrn migrate`, `dstrn migrate:rollback`, `dstrn migrate:status`)
- database seeder runner (`dstrn seed`, `dstrn seed:make`)
- parameter binding enforcement on all internal queries

<a name="view-engine-and-compiler"></a>

### view engine and compiler

- compiled `.d` template engine compiling template files directly into cached javascript modules
- template inheritance with `@extends`, `@section`, and `@yield`
- reusable partials via `@include` and component slots
- built in template directives: `@if`, `@elseif`, `@else`, `@foreach`, `@for`, `@forelse`, `@ruby`, `@t`, `@js`, `@json`
- server side javascript execution blocks (`@js ... @endjs`) for colocated view data computation
- automatic csrf token injection on all html form outputs

<a name="frontend-pipeline-and-dspa-router"></a>

### frontend pipeline and spa router

- lightweight client side spa router intercepting `<d-link>` navigation and `<d-form>` submissions with pushstate history
- declarative DOM event binding via `d-wire` and live update binding via `d-live`
- built in custom UI elements (`d-hold-button`, `d-modal`, `d-drawer`, `d-dropdown`, `d-combobox`, `d-file-input`, `d-color-picker`, `d-toggle`, `d-slider`, `d-skeleton`, `d-context-menu`)
- utility class styling system (`dstrn.css`) with 3 tier elevation (`bg-container-d`, `bg-container`, `bg-container-l`), contrast hierarchy, and responsive breakpoint prefixes
- build time asset minification and bundling pipeline

<a name="security-and-sessions"></a>

### security and sessions

- shield guard middleware providing automatic csrf token generation, cookie signing, and pow verification
- configurable rate limiting with ip based sliding windows and response headers
- bcrypt password hashing facade (`Hash.make()`, `Hash.check()`)
- session drivers: memory session driver, database session driver for distributed state, and stealth session driver for encrypted client state
- input validation engine

<a name="queues-and-task-scheduler"></a>

### queues and task scheduler

- worker thread background queue system with configurable concurrency and worker limits
- job dispatching facade (`Job.dispatch()`) with serialized payload passing
- task scheduler (`console/Schedule.js`) executing recurring commands and maintenance tasks

<a name="native-bridge-and-platform-runtimes"></a>

### native bridge and platform runtimes

- cross platform native bridge exposing unified `App.native` client APIs across web and native shells
- ios runtime shell built in swift using pure bounds windowing, webkit view, and native bridge dispatch
- android runtime shell built in kotlin using secure webview configuration and bridge interfaces
- desktop runtime shell built in rust with tauri and isolated webview security profiles
- zero config 4 file plugin architecture (`index.js`, `Plugin.swift`, `Plugin.kt`, `Plugin.rs`) with automatic web fallback
- built in core plugins: `auth` (keychain and keystore backing), `device` (hardware metadata), `storage` (secure persistent storage), `crypto` (ecdsa p256 webcrypto and hardware key generation), `audio` (native player pipeline), `notifications` (system notification center)

<a name="cli-and-architecture-analyzer"></a>

### cli and architecture analyzer

- global command line binary (`dstrn`)
- static architecture analyzer with visual dependency graph generation (`dstrn graph`), boundary enforcement (`dstrn lint`), dead code detection (`dstrn dead`), and project health scoring (`dstrn doctor`)
- translation parity validation (`dstrn lang:lint`) checking dictionaries and view keys
- interactive repl shell (`dstrn tinker`) with preloaded application context and model instances
- native development simulator and production builder (`dstrn simulate`, `dstrn build`)
- automatic deployment and certificate handling (`dstrn deploy`, `dstrn cert`)

---

<a name="historical-development"></a>

## historical development (2019 to 2026)

dframework began development in 2019 to eliminate tooling fragmentation and enforce consistency across both frontend and backend systems. between 2019 and 2026, the framework evolved through multiple foundational iterations prior to the 0.21 version.

<a name="development-milestones"></a>

### development milestones (v0.1.0 to v0.20.0)

- **2019 to 2020 (styling foundation and client helpers):** the framework originated as a client side styling system and utility layer. it contained javascript helpers, the baseline utility css definitions (which later formed `dstrn.css`), and the earliest custom frontend ui components. this version was used to build the original release of dstrn.xyz.
- **2020 to 2021 (frontend runtime and initial backend exploration):** core frontend components were refined, and the initial iteration of the client side single page application runtime was created. early exploratory work began on core backend capabilities (active record models, database abstractions, background queues, and native mobile stubs).
- **2021 to 2022 (spa stabilization and document database iteration):** the spa runtime reached a stable state with pushstate history navigation and dom swapping. a database adapter layer was implemented for document databases (mongodb). the styling system was refined with strict elevation and contrast rules. this iteration powered the closed release of dstrn.xyz.
- **2022 to 2023 (full framework transition and cli runtime):** active development shifted to building a unified full stack framework. the first versions of the `dstrn` cli, scaffolding tools, and development runtime were created. early routing mechanisms were introduced, though backend throughput and request dispatch remained uncompetitive with traditional high performance stacks.
- **2023 to 2024 (frontend optimization, standardization, and reactivity):** intensive performance tuning brought frontend render latency and dom mutation times to minimal levels. client runtime APIs were standardized, declarative reactivity bindings (`d-wire` and `d-live`) were introduced, and comprehensive frontend documentation was written.
- **2024 to 2025 (mysql migration, view compiler, and transport research):** the framework architecture aligned with its current structure. the router was rewritten with compiled regular expressions for low dispatch overhead, and the `.d` template compiler was built. mongodb was completely removed due to schema flexibility constraints, replaced by the modern mysql connection pool and active record orm layer with parameter bindings and migrations. the default node-http server being too restrictive, research and initial implementation of the custom native transport addon (`dfws`) commenced.
- **2025 to 2026 (transport benchmarking and micro optimizations):** thorough stress testing and benchmarking against competing frameworks were conducted. targeted micro optimizations were applied across all layers, active record query batching, and serialization pipelines, putting the framework in a highly competitive state in terms of throughput. the database layer, active record orm, worker queue, scheduler, and zero config native platform plugins (ios swift, android kotlin, desktop rust) reached feature completion.
- **2026 onwards (production hardening and release 0.21):** all subsystems were polished, tightly optimized, and unified into a closed world architecture, establishing the production baseline for v0.21.0.