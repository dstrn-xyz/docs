# cli

- [cli](#cli)

  - [introduction](#introduction)

  - [available commands](#available-commands)

    - [scaffolding](#scaffolding)
    - [database](#database)
    - [generators](#generators)
    - [system](#system)
    - [native](#native)
    - [architecture](#architecture)

  - [creating a project](#creating-a-project)

  - [running the server](#running-the-server)

  - [tinker](#tinker)

  - [user commands](#user-commands)

    - [creating a command](#creating-a-command)
    - [running a command](#running-a-command)

  - [logs management](#logs-management)

  - [architecture analyzer](#architecture-analyzer)

    - [dependency graph](#dependency-graph)
    - [architecture lint](#architecture-lint)
    - [dead code detection](#dead-code-detection)
    - [project doctor](#project-doctor)

  - [the compiler](#the-compiler)

    - [javascript compilation](#javascript-compilation)
    - [css compilation](#css-compilation)

<a name="introduction"></a>

## introduction

dframework includes a command line interface accessible via the `dstrn` binary. it provides commands for scaffolding new projects, managing the database, generating application files, analyzing architecture, and running an interactive shell.

```bash
dstrn help
```

<a name="available-commands"></a>

## available commands

<a name="scaffolding"></a>

### scaffolding

| command                   | description                                          |
| ------------------------- | ---------------------------------------------------- |
| `dstrn init <dir> [--ai]` | scaffold a minimal project with optional ai helpers  |
| `dstrn serve`             | run the server in the current directory              |
| `dstrn tinker`            | open an interactive repl shell with models preloaded |
| `dstrn key:generate`      | generate a random encryption key                     |

<a name="database"></a>

### database

| command                      | description                         |
| ---------------------------- | ----------------------------------- |
| `dstrn migrate`              | run all pending migrations          |
| `dstrn migrate:status`       | display migration status            |
| `dstrn migrate:rollback`     | rollback the last migration batch   |
| `dstrn migrate:make <table>` | create a new migration template     |
| `dstrn seed`                 | run all seeders                     |
| `dstrn seed:make <Name>`     | create a new seeder template        |
| `dstrn drop`                 | drop the database with confirmation |

<a name="generators"></a>

### generators

| command                        | description                  |
| ------------------------------ | ---------------------------- |
| `dstrn make:model <Name>`      | create a new model           |
| `dstrn make:controller <Name>` | create a new controller      |
| `dstrn make:middleware <Name>` | create a new middleware      |
| `dstrn make:command <Name>`    | create a new console command |
| `dstrn make:job <Name>`        | create a new background job  |
| `dstrn make:component <Name>`  | create a new view component  |

<a name="system"></a>

### system

| command                   | description                                     |
| ------------------------- | ----------------------------------------------- |
| `dstrn run <CommandName>` | run a user command from `console/commands`      |
| `dstrn logs:clear`        | clear or archive log files with date filtering  |
| `dstrn docs:publish`      | publish framework documentation to your project |
| `dstrn ai:publish`        | publish agentic documentation helpers           |

<a name="native"></a>

### native

| command                             | description                             |
| ----------------------------------- | --------------------------------------- |
| `dstrn simulate --<platform>`       | run in a platform simulator             |
| `dstrn build --target <platform>`   | build a production binary               |
| `dstrn make:plugin <Name>`          | scaffold a cross platform native plugin |
| `dstrn native:status`               | display native config and readiness     |
| `dstrn native:doctor`               | full environment diagnostic             |
| `dstrn native:clean [--<platform>]` | clean native build artifacts            |

<a name="architecture"></a>

### architecture

| command        | description                              |
| -------------- | ---------------------------------------- |
| `dstrn graph`  | display the application dependency graph |
| `dstrn lint`   | detect architecture violations           |
| `dstrn dead`   | find unreachable dead code               |
| `dstrn doctor` | compute a project quality score          |

<a name="creating-a-project"></a>

## creating a project

the `init` command scaffolds a new project with the standard directory structure, a default configuration file, a sample route, and a basic view.

```bash
dstrn init my-app
```

passing the `--ai` flag includes agentic documentation helpers in the generated project.

```bash
dstrn init my-app --ai
```

<a name="running-the-server"></a>

## running the server

the `serve` command starts the application server in the current directory.

```bash
dstrn serve
```

this loads the application configuration, imports all route files, boots the socket server, starts the scheduler, and begins listening for http requests. the startup banner displays the application url and port.

<a name="tinker"></a>

## tinker

the `tinker` command opens an interactive shell with the full application context loaded. all models are automatically imported and available as globals.

```bash
dstrn tinker
```

```
dframework> const users = await User.all();
dframework> users.length
42
dframework> await User.where('role', 'admin').get();
```

the shell persists command history to `.tinker_history` in your project root. all framework facades (`DB`, `Config`, `Session`, `Auth`, `Log`, etc.) are available in the context.

<a name="user-commands"></a>

## user commands

<a name="creating-a-command"></a>

### creating a command

scaffold a new command file using the `make:command` generator.

```bash
dstrn make:command PruneExpiredSessions
```

this creates `console/commands/PruneExpiredSessions.js`. the generated file exports a class with a `handle()` method where you write the command logic. the command has access to all framework globals.

```javascript
export default class PruneExpiredSessions {
  async handle() {
    const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    await DB.table('sessions').where('updated_at', '<', cutoff).delete();
    Log.info('expired sessions pruned');
  }
}
```

<a name="running-a-command"></a>

### running a command

execute any command from `console/commands` using the `run` command.

```bash
dstrn run PruneExpiredSessions
```

dframework imports the command class, instantiates it, calls `handle()`, and reports the elapsed execution time on completion.

commands are also used by the scheduler. any command referenced in `console/Schedule.js` is resolved from the same `console/commands` directory.

<a name="logs-management"></a>

## logs management

the `logs:clear` command provides flexible log file management with date range filtering, environment targeting, archiving, and a preview mode.

```bash
dstrn logs:clear
dstrn logs:clear --env=prod
dstrn logs:clear --before=2026-01-01
dstrn logs:clear --after=2026-05-01 --archive
dstrn logs:clear --preview
```

| flag                        | description                                       |
| --------------------------- | ------------------------------------------------- |
| `--env=prod` or `--env=dev` | target a specific environment log directory       |
| `--before=YYYY-MM-DD`       | clear logs before the specified date              |
| `--after=YYYY-MM-DD`        | clear logs after the specified date               |
| `--archive`                 | compress logs into a zip instead of deleting      |
| `--preview`                 | show which files would be affected without acting |

<a name="architecture-analyzer"></a>

## architecture analyzer

dframework includes a static analysis engine that builds a complete dependency graph of your application. it powers the `graph`, `lint`, `dead`, and `doctor` commands.

<a name="dependency-graph"></a>

### dependency graph

the `graph` command generates a visual representation of how your application is wired. it shows every http route and socket wire, the controllers they dispatch to, the middleware they pass through, and the models those controllers depend on.

```bash
dstrn graph
```

```
get /users (users.index)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ index()

post /users (users.store)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ store()

chat:message
 └─ AuthMiddleware
 └─ ChatController (Message)
     └─ onMessage()
```

for machine readable output, pass the `--json` flag.

```bash
dstrn graph --json
```

this outputs the full graph as a json object with `nodes` and `edges` arrays, suitable for external visualization tools.

<a name="architecture-lint"></a>

### architecture lint

the `lint` command detects structural violations in your application. it enforces the following dependency rules:

- controllers may not depend on other controllers
- models may not depend on controllers
- middleware may not depend on controllers
- routes may not depend directly on models

```bash
dstrn lint
```

if violations are found, each one is printed with a description.

```
 ERROR  controller UserController depends on controller AdminController
  controllers may not depend on controllers
```

<a name="dead-code-detection"></a>

### dead code detection

the `dead` command performs reachability analysis starting from all registered routes and socket wires. it traverses the dependency graph and reports any controllers, models, or middleware that are never referenced by any route.

```bash
dstrn dead
```

```
 ERROR  found 2 unreachable components

unused controller
  LegacyController

unused model
  TempModel
```

database tables are excluded from the analysis since they may be accessed through raw queries.

<a name="project-doctor"></a>

### project doctor

the `doctor` command computes a quality score from 0 to 100 based on three factors:

- **architecture violations** - each violation deducts 10 points, up to a maximum of 40
- **dead code** - each unreachable component deducts 5 points, up to a maximum of 30
- **complexity** - if the average number of edges per node exceeds 4, additional points are deducted based on the excess

```bash
dstrn doctor
```

```
 OK  health score 95 out of 100

deductions
  dead code minus 5 (1 unreachable components)
```

scores of 90 and above are displayed as success, 70-89 as informational, and below 70 as an error.

<a name="the-compiler"></a>

## the compiler

dframework includes a build time compiler that bundles and minifies the frontend runtime assets.

<a name="javascript-compilation"></a>

### javascript compilation

the js compiler concatenates the framework's core client side files (the main runtime, native bridge, custom elements, and utility functions), along with any user defined component files found in `public/js/components`. the combined output is minified with three compression passes, function name mangling, and toplevel mangling. the result is written to `public/js/dstrn.js`.

in the `local` environment with debug mode enabled, the `__DF_DEBUG__` global is set to `true`, enabling debug only code paths in the client runtime. in production, these paths are eliminated during dead code elimination.

<a name="css-compilation"></a>

### css compilation

the css compiler concatenates the framework's base styles, utility classes, icon font, and component styles. it then generates responsive variants for each utility class across six breakpoints (`sm`, `md`, `lg`, `xl`, `xxl`, `ultra`) using the `prefix\:classname` pattern. user defined component css files from `public/css/components` are included in the output.

the final output is minified by stripping comments, collapsing whitespace, removing unnecessary semicolons, and eliminating zero unit suffixes. the result is written to `public/css/dstrn.css`.
