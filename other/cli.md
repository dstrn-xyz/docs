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
  - [running tests](#running-tests)
  - [generating tests](#generating-tests)
  - [database advisor](#database-advisor)
  - [user commands](#user-commands)
  - [creating a command](#creating-a-command)
  - [running a command](#running-a-command)
  - [dispatching jobs](#dispatching-jobs)
  - [running queue workers](#running-queue-workers)
  - [logs management](#logs-management)
  - [deployment and certificates](#deployment-and-certificates)
    - [deploying applications](#deploying-applications)
    - [managing certificates](#managing-certificates)
  - [architecture analyzer](#architecture-analyzer)
    - [dependency graph](#dependency-graph)
    - [architecture lint](#architecture-lint)
    - [dead code detection](#dead-code-detection)
    - [project doctor](#project-doctor)
    - [translation lint](#translation-lint)
  - [the compiler](#the-compiler)
    - [javascript compilation](#javascript-compilation)
    - [css compilation](#css-compilation)

<a name="introduction"></a>

## introduction

dframework includes a command line interface accessible via the `dstrn` binary. it provides commands for scaffolding new projects, managing the database, generating application files, analyzing architecture, and running an interactive shell.

all commands except `dstrn init`, `dstrn help` (or `--help`), and `dstrn version` (or `-v`, `--version`) are gated and require execution within a valid dframework project directory containing `config/app.js`.

```bash
dstrn help
dstrn help serve
dstrn migrate --help
dstrn --version
```

passing a command name to `dstrn help <command>` or appending `--help` / `-h` to any command prints focused usage, arguments, and options for that specific command.

<a name="available-commands"></a>

## available commands

<a name="scaffolding"></a>

### scaffolding

| command               | description                                             |
| --------------------- | ------------------------------------------------------- |
| `dstrn init <dir>`    | scaffold a minimal project                              |
| `dstrn serve`         | run the server in the current directory                 |
| `dstrn test [target]` | run application tests via node test runner              |
| `dstrn tinker [code]` | open an interactive repl shell or execute code directly |
| `dstrn key:generate`  | generate a random encryption key                        |

<a name="database"></a>

### database

| command                      | description                                             |
| ---------------------------- | ------------------------------------------------------- |
| `dstrn migrate`              | run all pending migrations                              |
| `dstrn migrate:status`       | display migration status                                |
| `dstrn migrate:rollback`     | rollback the last migration batch                       |
| `dstrn migrate:make <name>`  | create a new migration template (infers table name from naming patterns) |
| `dstrn seed [name]`          | run database seeders                                    |
| `dstrn seed:make <Name>`     | create a new seeder template                            |
| `dstrn drop [--force]`       | drop all tables in non production environments with confirmation (or `--force`; always disabled in production) |
| `dstrn advisor [--apply] [--force]` | inspect unindexed relationships and generate migrations |

<a name="generators"></a>

### generators

| command                        | description                                                    |
| ------------------------------ | -------------------------------------------------------------- |
| `dstrn make:model <Name>`      | create a new model                                             |
| `dstrn make:controller <Name>` | create a new controller                                        |
| `dstrn make:middleware <Name>` | create a new middleware                                        |
| `dstrn make:command <Name>`    | create a new console command                                   |
| `dstrn make:job <Name>`        | create a new background job                                    |
| `dstrn make:component <Name>`  | create a new client web component (`dComponent`) in `public/js/components` |
| `dstrn make:test <Name>`       | create a new test file (`--feature`, `--browser`, `--unit`)                |

<a name="system"></a>

### system

| command                                 | description                                                                   |
| --------------------------------------- | ----------------------------------------------------------------------------- |
| `dstrn run <CommandName>`               | run a user command from `console/commands`                                    |
| `dstrn queue:work [--queue=] [--driver=] [--once]` | start processing background queue jobs as a daemon                            |
| `dstrn dispatch <JobName> [--payload=] [--tracking-id=]` | run a job directly in the current process with optional payload and correlation id |
| `dstrn logs:clear`                     | clear or archive log files with date filtering                                |
| `dstrn deploy`                         | generate production deployment and service configurations                     |
| `dstrn cert <action>`                  | manage tls certificates (`obtain`, `renew`, `status`)                         |
| `dstrn cert:obtain`                    | obtain let's encrypt ssl certificates                                         |
| `dstrn cert:renew`                     | renew expiring ssl certificates                                               |
| `dstrn cert:status`                    | display certificate validity and expiration details                           |
| `dstrn docs:publish`                   | publish framework documentation to your project                               |

<a name="native"></a>

### native

| command                             | description                             |
| ----------------------------------- | --------------------------------------- |
| `dstrn simulate --<platform>`       | run in a platform simulator             |
| `dstrn build --<platform>`          | build a production binary               |
| `dstrn make:plugin <Name>`          | scaffold a cross platform native plugin |
| `dstrn native:status`               | display native config and readiness     |
| `dstrn native:doctor [--<platform>]` | check native environment (`--desktop`, `--android`, or `--ios` to require a target) |
| `dstrn native:clean [--<platform>]` | clean native build artifacts            |

<a name="architecture"></a>

### architecture

| command           | description                              |
| ----------------- | ---------------------------------------- |
| `dstrn graph`     | display the application dependency graph |
| `dstrn lint`      | detect architecture violations           |
| `dstrn dead`      | find unreachable dead code               |
| `dstrn doctor`    | compute a project quality score          |
| `dstrn lang:lint` | check translation parity and view keys   |

<a name="creating-a-project"></a>

## creating a project

the `init` command scaffolds a new project with the standard directory structure, a default configuration file, a sample route, and a basic view.

```bash
dstrn init my-app
```

<a name="running-the-server"></a>

## running the server

the `serve` command starts the application server in the current directory.

```bash
dstrn serve
```

this loads the application configuration, imports all route files, boots the socket server, starts the scheduler, and begins listening for http requests. the startup banner displays the application url and port.

you can specify a custom host or port using `--host` and `--port`:

```bash
dstrn serve --host=0.0.0.0 --port=3000
```

providing `--host` as a standalone flag without a value binds the server to `0.0.0.0` to expose it to the local network:

```bash
dstrn serve --host
```

<a name="tinker"></a>

## tinker

the `tinker` command provides an interactive repl shell or executes inline code directly with the full application context and all models preloaded.

### interactive shell

start an interactive session by running the command without arguments:

```bash
dstrn tinker
```

```
dframework> const users = await User.all();
dframework> users.length
42
dframework> await User.where('role', 'admin').get();
```

the shell persists command history to `.dframework_history` in your project root. all framework facades (`DB`, `Config`, `Session`, `Auth`, `Log`, etc.) and models are available in the context.

### inline code execution

pass code directly as a positional argument or use the `--execute` (`-e`) flag to evaluate code and output the formatted result immediately without starting an interactive session:

```bash
dstrn tinker "await User.count()"
dstrn tinker "await DB.table('users').where('status', 'active').get()"
dstrn tinker --execute="await User.find(1)"
dstrn tinker -e "const u = await User.first(); return u.email;"
```

inline execution supports top level await expressions and multi statement blocks. all models in `models/` are loaded into global scope before execution starts.

<a name="running-tests"></a>

## running tests

the `test` command runs your test suites using node's built in test runner with automatic environment isolation (`APP_ENV=testing`), discovering all test files matching `*.test.js` under the `tests/` directory. running tests strictly requires a `.env.testing` file in the project root:

```bash
dstrn test
dstrn test tests/feature/UserTest.test.js
dstrn test --watch
dstrn test --only
dstrn test --raw
dstrn test --reporter=./custom-reporter.js
```

<a name="generating-tests"></a>

## generating tests

the `make:test` command scaffolds a test file structured with appropriate imports and lifecycle assertions:

```bash
dstrn make:test UserTest
dstrn make:test LoginTest --browser
dstrn make:test CalculatorTest --unit
```

<a name="database-advisor"></a>

## database advisor

the `advisor` command inspects your database schema, column definitions, and active query history to detect missing indexes on foreign keys, junction tables, and frequently filtered column pairs.

```bash
dstrn advisor
```

```
 INFO  database index advisor

  tracks .....................................................  WARN  [idx_tracks_album_id]
    columns: album_id
    reason:  foreign key column album_id lacks leading index

  track_artists ..............................................  WARN  [idx_track_artists_track_id_artist_id]
    columns: track_id, artist_id
    reason:  frequently filtered column pair (track_id, artist_id) lacks composite index

 OK  found 2 recommendations (run "dstrn advisor --apply" to generate and execute migrations) (12.30ms)
```

to automatically scaffold and execute timestamped migrations with all recommended indexes, pass the `--apply` flag:

```bash
dstrn advisor --apply
```

```
 INFO  database index advisor

  tracks .....................................................  WARN  [idx_tracks_album_id]
    columns: album_id
    reason:  foreign key column album_id lacks leading index

  track_artists ..............................................  WARN  [idx_track_artists_track_id_artist_id]
    columns: track_id, artist_id
    reason:  frequently filtered column pair (track_id, artist_id) lacks composite index

  2026_10_28_120000_add_idx_tracks_album_id.js .................  OK  [generated]
  2026_10_28_120001_add_idx_track_artists_track_id_artist_id.js ..  OK  [generated]

 INFO  running migrations

  2026_10_28_120000_add_idx_tracks_album_id.js .................  OK  [batch: 1] (4.50ms)
  2026_10_28_120001_add_idx_track_artists_track_id_artist_id.js ..  OK  [batch: 1] (3.80ms)

 OK  migrated 2 migrations (10.20ms)

 OK  all migrations applied successfully (18.50ms)
```

each generated migration file exports `up` and `down` functions with `table.index()` and `table.dropIndex()` calls, allowing full rollback support via `dstrn migrate:rollback`.

<a name="user-commands"></a>

## scheduled tasks and manual execution

you can create scheduler commands that live in `console/commands`. these tasks run automatically on configured schedules and can also be executed manually on demand using `dstrn run`.

<a name="creating-a-command"></a>

### creating a command

scaffold a new command file using the `make:command` generator:

```bash
dstrn make:command PruneExpiredSessions
```

this creates `console/commands/PruneExpiredSessionsCommand.js` (the generator automatically appends the `Command` suffix if omitted). the generated file exports a default class with an `async handle(args, options)` method where you write the execution logic. the task has direct access to all framework globals (`DB`, `Log`, `Config`, models):

```javascript
export default class PruneExpiredSessionsCommand {
  async handle(args, options) {
    const isDryRun = this.hasOption('dry-run');
    const batchSize = parseInt(this.option('batch', 500), 10);
    const targetTable = this.argument(0, 'sessions');

    if (isDryRun) {
      Log.info(`previewing session cleanup for ${targetTable} in batches of ${batchSize}`);
      return;
    }

    const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    await DB.table(targetTable).where('updated_at', '<', cutoff).delete();
    Log.info('expired sessions pruned');
  }
}
```

<a name="running-a-command"></a>

### running a command

you can execute any command on demand from the terminal using the `run` command:

```bash
dstrn run PruneExpiredSessions
```

file names ending with `.js` are also accepted to facilitate shell tab completion:

```bash
dstrn run PruneExpiredSessions.js
```

to pass parameters when running manually, include them after the task name. options and flags can be passed using either `--key value` or `--key=value` format, alongside positional arguments:

```bash
dstrn run PruneExpiredSessions sessions --batch=1000 --dry-run
# or
dstrn run PruneExpiredSessions sessions --batch 1000 --dry-run
```

dframework automatically parses the cli arguments and injects helper methods onto the command instance:

| method / property                | description                                                          |
| -------------------------------- | -------------------------------------------------------------------- |
| `this.option('name', fallback)`  | retrieves an option value from `--name value` or `--name=value`      |
| `this.hasOption('name')`         | returns boolean true if a flag like `--dry-run` was passed           |
| `this.argument(index, fallback)` | retrieves a positional argument by zero-based index                  |
| `this.options`                   | dictionary containing all parsed options and flags                   |
| `this.arguments`                 | array containing all positional arguments                            |
| `options` (2nd parameter)        | the parsed options object passed directly to `handle(args, options)` |

dframework imports the class from `console/commands/PruneExpiredSessionsCommand.js` (or `PruneExpiredSessions.js`), instantiates it, calls `handle(args, options)`, and reports the elapsed execution time on completion.

tasks in `console/commands` are also used by the scheduler. when scheduled in `console/Schedule.js`, the scheduler calls `handle([], {})` where `this.option()` safely returns the configured fallback values.

<a name="dispatching-jobs"></a>

### dispatching jobs

you can execute jobs directly on demand from the command line using `dstrn dispatch` (runs synchronously in the current process without queuing):

```bash
dstrn dispatch CleanupExpiredSessionsJob
```

file names ending with `.js` are also accepted to facilitate shell tab completion:

```bash
dstrn dispatch CleanupExpiredSessionsJob.js
```

you can pass a json payload using the `--payload` flag:

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}'
```

you can also assign an explicit correlation identifier using the `--tracking-id` flag:

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}' --tracking-id 'custom-correlation-id'
```

<a name="running-queue-workers"></a>

### running queue workers

you can run a dedicated worker daemon to process background jobs from the queue:

```bash
dstrn queue:work
```

you can filter by queue, override the driver, or process a single job and exit:

```bash
dstrn queue:work --queue=emails
dstrn queue:work --driver=database
dstrn queue:work --once
```

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
| `--archive`                 | compress each matched log into a `.gz` file inside an `archives` subdirectory and remove the original |
| `--preview`                 | show which files would be affected without acting |

<a name="deployment-and-certificates"></a>

## deployment and certificates

<a name="deploying-applications"></a>

### deploying applications

the `deploy` command generates production configurations for bare metal deployment with native tls (default), reverse proxy with caddy (`--proxy`), or edge routing with cloudflare (`--edge`).

```bash
dstrn deploy --domain=api.example.com --email=admin@example.com
dstrn deploy --proxy --domain=api.example.com
dstrn deploy --edge --domain=api.example.com
dstrn deploy --domain=api.example.com --email=admin@example.com --dry-run
sudo dstrn deploy --domain=api.example.com --email=admin@example.com --install
```

| flag                 | description                                                                   |
| -------------------- | ----------------------------------------------------------------------------- |
| `--domain=<domains>` | comma separated target domains                                                |
| `--email=<email>`    | acme registration email for let's encrypt certificates                        |
| `--proxy`            | generate caddy reverse proxy configuration instead of direct tls              |
| `--edge`             | generate plain http configuration with cloudflare guidance                    |
| `--port=<port>`      | custom application port (default 8250 or 443 for direct tls)                  |
| `--user=<user>`      | system user for systemd service (default current user)                        |
| `--dir=<dir>`        | working directory path                                                        |
| `--dry-run`          | preview generated configurations in console without writing to disk           |
| `--install`          | write configuration files directly to system paths (requires root privileges) |

<a name="managing-certificates"></a>

### managing certificates

the `cert` command family handles certificate issuance, renewals, and status monitoring.

```bash
dstrn cert:obtain --domain=api.example.com --email=admin@example.com
dstrn cert:obtain --domain=api.example.com --email=admin@example.com --staging
dstrn cert:renew --domain=api.example.com --email=admin@example.com
dstrn cert:status
```

| command                                      | description                                                   |
| -------------------------------------------- | ------------------------------------------------------------- |
| `dstrn cert <obtain\|renew\|status> [--dir=...]` | manage tls certificates (`obtain`, `renew`, or `status`)      |
| `dstrn cert:obtain --domain=... --email=... [--dir=...]` | obtain a new let's encrypt ssl certificate                    |
| `dstrn cert:renew --domain=... --email=... [--dir=...]`  | renew certificates expiring within 30 days                    |
| `dstrn cert:status [--domain=...] [--dir=...]` | show validity and expiration dates for installed certificates |

<a name="architecture-analyzer"></a>

## architecture analyzer

dframework includes a static analysis engine that builds a complete dependency graph of your application. it powers the `graph`, `lint`, `dead`, `doctor`, and `lang:lint` commands.

<a name="dependency-graph"></a>

### dependency graph

the `graph` command generates a visual representation of how your application is wired. it shows every http route and socket wire, the controllers they dispatch to, the middleware they pass through, and the models those controllers depend on.

```bash
dstrn graph
```

```
 INFO  dependency graph of how the application is wired

http routes
get /users (users.index)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ index()

post /users (users.store)
 └─ AuthMiddleware
 └─ UserController (User)
     └─ store()

socket routes
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

the `lint` command detects structural violations in your application.

> [!IMPORTANT]
> `dstrn lint` enforces strict architectural boundaries: controllers may not depend on other controllers, models may not depend on controllers, middleware may not depend on controllers, routes may not depend directly on models, and jobs should not be imported directly (dispatch via Job.dispatch instead).

```bash
dstrn lint
```

if violations are found, each one is printed with a description.

```
 INFO  architecture lint

  UserController (controller) ................................  ERROR  [AdminController]
    controllers may not depend on controllers

 ERROR  found 1 architecture violation (12.30ms)
```

<a name="dead-code-detection"></a>

### dead code detection

the `dead` command performs reachability analysis starting from all registered routes, socket wires, and scheduled tasks (`console/Schedule.js`). it traverses the dependency graph and reports any controllers, models, jobs, or middleware that are never referenced.

```bash
dstrn dead
```

```
 INFO  dead code analysis

  LegacyController ...........................................  WARN  [controller]
  TempModel ..................................................  WARN  [model]

 ERROR  found 2 unreachable components (15.20ms)
```

database tables are excluded from the analysis since they may be accessed through raw queries.

<a name="project-doctor"></a>

### project doctor

the `doctor` command computes a quality score from 0 to 100 based on three factors:

- **architecture violations** (each violation deducts 10 points, up to a maximum of 40)
- **dead code** (each unreachable component deducts 5 points, up to a maximum of 30)
- **complexity** (if the average number of edges per node exceeds 4, additional points are deducted based on the excess)

```bash
dstrn doctor
```

```
 INFO  project architecture health

  architecture structure .....................................  OK  [no violations]
  code reachability ..........................................  WARN  [-5] (1 unreachable)
  project complexity .........................................  OK  [2.1 edges per node]

 OK  health score 95 out of 100 (10.50ms)
```

scores of 70 and above indicate passing health, and scores below 70 are reported as an error.

<a name="translation-lint"></a>

### translation lint

the `lang:lint` command verifies translation keys across all configured languages and validates references throughout your views and code. it checks for two categories of issues:

- **asymmetric keys** (keys defined in one locale dictionary but missing from another)
- **missing references** (translation keys used in `.d` views or controllers with `@t()` or `t()` that do not exist in the translation files)

```bash
dstrn lang:lint
```

if issues are found, the command reports the exact files, line numbers, and missing locales:

```
 INFO  translation verification

missing translations:
  views/index.d:4 ............................................  ERROR  'ui.tagline' missing in [ja]

asymmetric translation keys:
  ja: ui.tagline .............................................  WARN  defined in [en]

 ERROR  found 2 translation issues (14.20ms)
```

if all translations are complete and match view usage, a success status is returned.

pass `--strict` to fail with a non zero exit code if unused translation keys exist in dictionaries:

```bash
dstrn lang:lint --strict
```

<a name="the-compiler"></a>

## the compiler

dframework includes a build time compiler that bundles and minifies the frontend runtime assets.

<a name="javascript-compilation"></a>

### javascript compilation

the js compiler concatenates the framework's core client side files (the main runtime, native bridge, custom elements, and utility functions), along with any user defined component files found in `public/js/components`. the combined output is minified with three compression passes, function name mangling, and toplevel mangling. the result is written to `storage/framework/public/js/dstrn.js`.

in the `local` environment with debug mode enabled, the `__DF_DEBUG__` global is set to `true`, enabling debug only code paths in the client runtime. in production, these paths are eliminated during dead code elimination.

<a name="css-compilation"></a>

### css compilation

the css compiler concatenates the framework's base styles, utility classes, icon font, and component styles. it then generates responsive variants for each utility class across six breakpoints (`sm`, `md`, `lg`, `xl`, `xxl`, `ultra`) using the `prefix\:classname` pattern. user defined component css files from `public/css/components` are included in the output.

the final output is minified by stripping comments, collapsing whitespace, removing unnecessary semicolons, and eliminating zero unit suffixes. the result is written to `storage/framework/public/css/dstrn.css`.