# logging and errors

- [logging and errors](#logging-and-errors)
  - [introduction](#introduction)
  - [the logger](#the-logger)
    - [log levels](#log-levels)
    - [writing log messages](#writing-log-messages)
    - [table logging](#table-logging)
  - [log output](#log-output)
    - [console output](#console-output)
    - [file output](#file-output)
    - [log file rotation](#log-file-rotation)
  - [the global log facade](#the-global-log-facade)
  - [debug bar integration](#debug-bar-integration)
  - [configuration](#configuration)
  - [the error handler](#the-error-handler)
    - [error pages](#error-pages)
    - [custom error pages](#custom-error-pages)
    - [debug error page](#debug-error-page)
    - [error tips](#error-tips)
  - [stack trace classification](#stack-trace-classification)
    - [frame categories](#frame-categories)
    - [view line mapping](#view-line-mapping)
    - [hot reload path resolution](#hot-reload-path-resolution)

<a name="introduction"></a>

## introduction

dframework provides a dual channel logging system that writes to both the console and persistent log files. it includes an error handler that renders error pages, classifies stack traces by component type, maps compiled view errors back to their original template lines, and provides diagnostic tips for common mistakes.

<a name="the-logger"></a>

## the logger

<a name="log-levels"></a>

### log levels

the logger supports six log levels, each with its own color coding and behavior.

| level   | color   | purpose                                                            |
| ------- | ------- | ------------------------------------------------------------------ |
| `debug` | cyan    | development only messages, suppressed unless debug mode is enabled |
| `info`  | blue    | general informational messages                                     |
| `hot`   | orange  | hot reload events during local development                         |
| `warn`  | yellow  | non critical warnings                                              |
| `error` | red     | errors and exceptions                                              |
| `table` | magenta | structured tabular data output                                     |

the minimum log level is determined by whether debug mode is enabled. when `app.debug` is `true`, all levels including `debug` are output. when debug is disabled, `debug` messages are suppressed.

<a name="writing-log-messages"></a>

### writing log messages

each log level has a corresponding method accessible via the global `Log` facade. all methods accept variadic arguments that are automatically formatted.

```javascript
Log.debug('cache miss for key:', cacheKey);
Log.info('user registered:', user.email);
Log.warn('rate limit approaching for ip:', clientIp);
Log.error('payment failed:', error);
```

the logger formats different argument types. strings are output as is, `Error` objects display their full stack trace, promises display as `Promise { <pending> }`, and objects are pretty printed with a depth of 3 and color support in the local environment.

<a name="table-logging"></a>

### table logging

the `Log.table()` method renders an array of objects as a formatted ascii table with box drawing characters.

```javascript
app.logger.table([
  { name: 'users', rows: 1250, size: '2.4 MB' },
  { name: 'posts', rows: 8340, size: '15.1 MB' },
  { name: 'sessions', rows: 89, size: '0.1 MB' }
], 'database overview');
```

```
database overview
┌─────────┬──────┬─────────┐
│ name    │ rows │ size    │
├─────────┼──────┼─────────┤
│ users   │ 1250 │ 2.4 MB  │
│ posts   │ 8340 │ 15.1 MB │
│ sessions│ 89   │ 0.1 MB  │
└─────────┴──────┴─────────┘
```

<a name="log-output"></a>

## log output

<a name="console-output"></a>

### console output

in the `local` environment, all log messages are output to the console with ansi color coding. error and warning messages use `console.error` to separate them from standard output.

in the `production` environment, colors are stripped and only `warn` and `error` messages go to `console.error`. all other levels use `console.log`.

<a name="file-output"></a>

### file output

every log message is simultaneously written to a persistent log file. log files are stored in the `logs` directory at the root of your project, separated by environment.

```
logs/
├── dev/
│   ├── dframework_2026-06-04.log
│   └── dframework_2026-06-05.log
└── prod/
    ├── dframework_2026-06-04.log
    └── dframework_2026-06-05.log
```

the `local` environment writes to `logs/dev/` and production writes to `logs/prod/`. ansi color codes are automatically stripped from file output.

<a name="log-file-rotation"></a>

### log file rotation

log files are automatically rotated daily. each file is named with the date (`dframework_YYYY-MM-DD.log`). when the date changes, the logger detects that the current file path no longer matches, closes the previous write stream, and opens a new one for the current date.

you can clear log files using the cli.

```bash
dstrn logs:clear
dstrn logs:clear --env=prod
dstrn logs:clear --before=2026-01-01
dstrn logs:clear --after=2026-05-01 --archive
dstrn logs:clear --preview
```

<a name="debug-bar-integration"></a>

## debug bar integration

in the `local` environment with debug mode enabled, log messages are automatically captured by the request context and included in the debug bar response. each log entry records its level, message (with colors stripped), and timestamp. this allows you to see every log message that was generated during a specific request directly in the browser's debug panel.

<a name="configuration"></a>

## configuration

the logger reads its configuration from the application config.

| config key  | effect                                                                        |
| ----------- | ----------------------------------------------------------------------------- |
| `app.env`   | determines the log file directory (`dev` vs `prod`) and console color mode    |
| `app.debug` | when `true`, enables the `debug` log level and activates the debug error page |

```javascript
// config/app.js
export default {
  env: 'local',
  debug: true
};
```

<a name="the-error-handler"></a>

## the error handler

the `ErrorHandler` class manages error rendering and logging throughout the application. it is instantiated during app boot and caches all error page templates for instant rendering.

<a name="error-pages"></a>

### error pages

dframework includes built in error pages for the following http status codes: `400`, `401`, `403`, `404`, `405`, `419`, `429`, `500`, `502`, `503`, and `504`. each page uses the framework's view template syntax and displays a styled error screen to the user.

when an error occurs, the error handler selects the appropriate template based on the status code. if no template exists for the specific code, it falls back to the `500` template. if no templates are available at all, a plain text response is sent.

<a name="custom-error-pages"></a>

### custom error pages

you can override any built in error page by placing a file with the same name in your `views/errors` directory. user error pages take priority over the framework defaults.

```
views/errors/
├── 404.d
├── 500.d
└── 503.d
```

your custom error pages have access to the `{{ status }}` placeholder which is replaced with the numeric status code.

<a name="debug-error-page"></a>

### debug error page

when `app.debug` is `true` and the error is not a 404, the framework renders a detailed debug error page instead of the production error page. the debug page displays:

- the http status code
- the error message
- the classified stack trace with component level color coding

this page is designed for development only and is never shown in the production environment.

<a name="error-tips"></a>

### error tips

the error handler includes a pattern matching system that recognizes common error messages and provides diagnostic tips. when an error is logged, the handler checks the message against a list of known patterns.

| error pattern                         | tip                                                                  |
| ------------------------------------- | -------------------------------------------------------------------- |
| `Unexpected reserved word`            | check for missing async keyword before method definitions            |
| `Cannot read properties of undefined` | verify object exists before accessing properties                     |
| `is not a function`                   | verify the method exists and is properly imported                    |
| `Cannot find module`                  | check import path and ensure file exists                             |
| `await is only valid in async`        | add async keyword to the function declaration                        |
| `EADDRINUSE`                          | port is already in use, stop other processes or use a different port |
| `ER_NO_SUCH_TABLE`                    | run migrations to create the database table                          |
| `ER_BAD_FIELD_ERROR`                  | column does not exist in table, check schema or migration            |
| `ER_DUP_ENTRY`                        | duplicate value for unique constraint, check data or schema          |

tips are displayed in cyan below the error message in the console output.

```
[Socket] Template rendering failed: Cannot read properties of undefined (reading 'name')
tip: verify object exists before accessing properties
  ▸ [controller]   UserController                  controllers/UserController.js:42
  ▸ [view]         home                            views/home.d:15
```

<a name="stack-trace-classification"></a>

## stack trace classification

the error handler transforms raw stack traces into classified, color coded frames that highlight the most relevant parts of the trace.

<a name="frame-categories"></a>

### frame categories

each stack frame is classified into a category based on its file path.

| category     | color   | matched path                     |
| ------------ | ------- | -------------------------------- |
| `controller` | yellow  | `/controllers/`                  |
| `middleware` | blue    | `/middlewares/`                  |
| `model`      | green   | `/models/`                       |
| `view`       | magenta | `view:///`                       |
| `route`      | yellow  | `/routes/`                       |
| `job`        | cyan    | `/jobs/`                         |
| `command`    | cyan    | `/console/`                      |
| `app`        | white   | anything within the project root |

frames from `node_modules` and the framework's core directory are automatically filtered out. only your application code is shown in the trace.

```
  ▸ [controller]   register                        controllers/AuthController.js:28
  ▸ [model]        create                          models/User.js:15
  ▸ [view]         auth.register                   views/auth/register.d:42
```

<a name="view-line-mapping"></a>

### view line mapping

when an error originates from a compiled view template, the error handler maps the compiled javascript line number back to the original line in the `.d` template file. this is done using the line map generated during view compilation, ensuring that the stack trace points to the exact line in your template that caused the error rather than the generated code.

<a name="hot-reload-path-resolution"></a>

### hot reload path resolution

in the `local` environment, the framework uses cache busted file paths (`.hot-*`) for hot reloading. the error handler resolves these back to the original file paths in the stack trace, so you always see clean, recognizable paths.