# scheduling and commands

- [scheduling and commands](#scheduling-and-commands)

  - [introduction](#introduction)

  - [defining schedules](#defining-schedules)

    - [the schedule file](#the-schedule-file)
    - [scheduling commands](#scheduling-commands)
    - [scheduling closures](#scheduling-closures)

  - [frequency options](#frequency-options)

    - [preset intervals](#preset-intervals)
    - [custom intervals](#custom-intervals)

  - [creating commands](#creating-commands)

    - [the command class](#the-command-class)
    - [generating commands](#generating-commands)
    - [running commands](#running-commands)

  - [how the scheduler works](#how-the-scheduler-works)

    - [automatic startup](#automatic-startup)
    - [execution and rescheduling](#execution-and-rescheduling)
    - [manual execution](#manual-execution)

  - [error handling](#error-handling)

<a name="introduction"></a>

## introduction

dframework includes a built in task scheduler that lets you define recurring background tasks directly in your application code. instead of configuring system level cron jobs on your server, you define your schedule in a single file and the framework handles the rest. tasks can be references to command classes or inline functions.

the scheduler starts automatically when the application boots and runs for the lifetime of the process. each task is executed in the main thread synchronously in an isolated context and automatically rescheduled after completion.

<a name="defining-schedules"></a>

## defining schedules

<a name="the-schedule-file"></a>

### the schedule file

your schedule is defined in `console/Schedule.js` at the root of your project. this file exports a default function that receives the `Scheduler` instance. you use the scheduler's `command()` method to register tasks and chain a frequency method.

```javascript
export default function (scheduler) {
  scheduler.command('CleanupExpiredSessions').daily();
  scheduler.command('SyncExternalData').everyFifteenMinutes();
  scheduler.command('GenerateSitemaps').weekly();
}
```

the framework automatically detects this file on startup and begins executing the defined schedule.

<a name="scheduling-commands"></a>

### scheduling commands

when you pass a string to `scheduler.command()`, the scheduler resolves it as a command file in your `console/commands` directory. the file must export a default class with a `handle()` method.

```javascript
scheduler.command('PruneOldLogs').daily();
```

this will look for `console/commands/PruneOldLogs.js`, instantiate the class, and call its `handle()` method at the specified interval.

<a name="scheduling-closures"></a>

### scheduling closures

for simple tasks that don't warrant a dedicated command file, you can pass an inline function directly.

```javascript
export default function (scheduler) {
  scheduler.command(async () => {
    const count = await Session.where('expires_at', '<', new Date()).delete();
    Log.info(`pruned ${count} expired sessions`);
  }).everyThirtyMinutes();
}
```

<a name="frequency-options"></a>

## frequency options

<a name="preset-intervals"></a>

### preset intervals

the scheduler provides a set of human readable preset interval methods that you chain after `command()`.

| method                  | interval   |
| ----------------------- | ---------- |
| `everyMinute()`         | 60 seconds |
| `everyFiveMinutes()`    | 5 minutes  |
| `everyTenMinutes()`     | 10 minutes |
| `everyFifteenMinutes()` | 15 minutes |
| `everyThirtyMinutes()`  | 30 minutes |
| `hourly()`              | 1 hour     |
| `daily()`               | 24 hours   |
| `weekly()`              | 7 days     |

```javascript
scheduler.command('HeartbeatCheck').everyMinute();
scheduler.command('CacheWarmer').hourly();
scheduler.command('WeeklyDigest').weekly();
```

<a name="custom-intervals"></a>

### custom intervals

for intervals not covered by the presets, use the `every()` method with a duration in milliseconds.

```javascript
scheduler.command('PollExternalApi').every(45000);
scheduler.command('RotateApiKeys').every(12 * 60 * 60 * 1000);
```

<a name="creating-commands"></a>

## creating commands

<a name="the-command-class"></a>

### the command class

a command is a class stored in the `console/commands` directory. it must export a default class with a `handle()` method. the `handle` method contains the logic that will be executed when the command runs.

```javascript
export default class PruneOldLogsCommand {
  async handle() {
    const cutoff = new Date();
    cutoff.setDate(cutoff.getDate() - 30);
    await DB.table('logs').where('created_at', '<', cutoff).delete();
    Log.info('old logs pruned');
  }
}
```

commands used by the scheduler do not receive any arguments. if you need to pass data, use configuration values or database lookups inside the command itself.

<a name="generating-commands"></a>

### generating commands

you can scaffold a new command file using the cli.

```bash
dstrn make:command PruneOldLogs
```

this creates `console/commands/PruneOldLogs.js` with the correct boilerplate structure.

<a name="running-commands"></a>

### running commands

you can execute any command on demand using the `run` cli command.

```bash
dstrn run PruneOldLogs
```

dframework imports the command class from `console/commands/PruneOldLogs.js`, instantiates it, calls `handle()`, and reports the elapsed time on completion.

<a name="how-the-scheduler-works"></a>

## how the scheduler works

<a name="automatic-startup"></a>

### automatic startup

when the application starts, the framework checks for a `console/Schedule.js` file. if it exists, the framework imports it, passes a new `Scheduler` instance, and calls `scheduler.start()` after all tasks are registered. each registered task is executed immediately on startup and then rescheduled based on its configured interval.

<a name="execution-and-rescheduling"></a>

### execution and rescheduling

when a task's interval elapses, the scheduler calls the task's handler (either a command class or an inline function). after the task completes, the scheduler calculates the next execution time by subtracting the elapsed execution time from the interval, ensuring consistent spacing between runs even if the task takes a significant amount of time to complete. the next run is scheduled using `setTimeout`.

each task tracks its `lastRun` timestamp. the scheduler uses this to compute the appropriate delay before the next execution.

<a name="manual-execution"></a>

### manual execution

you can programmatically trigger any registered scheduled task by name using `scheduler.run()`.

```javascript
await app.scheduler.run('CleanupExpiredSessions');
```

this immediately executes the task and resets its scheduling timer.

<a name="error-handling"></a>

## error handling

if a scheduled task throws an error, the scheduler catches it and logs the error using the application logger. the task is then rescheduled normally. a single failing task does not affect other scheduled tasks or the application's http server.

```
[Scheduler] PruneOldLogs caused error: ER_NO_SUCH_TABLE: Table 'logs' doesn't exist
```

if the task itself causes an unexpected error during the scheduling phase, a separate catch logs it and prevents the error from propagating to the main process.
