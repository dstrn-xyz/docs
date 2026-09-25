# jobs and queues

- [jobs and queues](#jobs-and-queues)
  - [introduction](#introduction)
  - [creating jobs](#creating-jobs)
    - [the job class](#the-job-class)
    - [generating jobs](#generating-jobs)
  - [dispatching jobs](#dispatching-jobs)
    - [from controllers](#from-controllers)
    - [from anywhere](#from-anywhere)
    - [from the command line](#from-the-command-line)
    - [payload data](#payload-data)
  - [the job facade](#the-job-facade)
  - [how the worker pool works](#how-the-worker-pool-works)
    - [worker isolation](#worker-isolation)
    - [concurrency limits](#concurrency-limits)
    - [automatic queuing](#automatic-queuing)
  - [job lifecycle](#job-lifecycle)
    - [bootstrapping](#bootstrapping)
    - [database access](#database-access)
    - [broadcasting from jobs](#broadcasting-from-jobs)
    - [error handling](#error-handling)
  - [configuration](#configuration)

<a name="introduction"></a>

## introduction

dframework includes a native background job queue that runs jobs in isolated worker threads. jobs are dispatched from your application code and executed asynchronously without blocking the main http process. this is ideal for sending emails, processing images, syncing external apis, or any task that should not delay the user's response.

> [!NOTE]
> the queue system requires no external dependencies like redis or a separate queue server. it runs entirely within the node.js runtime using the built in `worker_threads` module.

<a name="creating-jobs"></a>

## creating jobs

<a name="the-job-class"></a>

### the job class

a job is a class stored in the `jobs` directory at the root of your project. every job must export a default class with a `handle(payload)` method. the `handle` method receives the payload that was passed when the job was dispatched and contains the actual work to be performed.

```javascript
export default class SendWelcomeEmailJob {
  async handle(payload) {
    const user = await User.find(payload.userId);
    await Mailer.send(user.email, 'welcome', { name: user.name });
  }
}
```

the `handle` method can be asynchronous. the worker waits for the returned promise to resolve before reporting completion.

<a name="generating-jobs"></a>

### generating jobs

you can scaffold a new job file using the cli.

```bash
dstrn make:job SendWelcomeEmailJob
```

this creates a `jobs/SendWelcomeEmailJob.js` file with the correct boilerplate structure.

<a name="dispatching-jobs"></a>

## dispatching jobs

<a name="from-controllers"></a>

### from controllers

the most common way to dispatch a job is from a controller after handling the user's request.

```javascript
export default class RegistrationController {
  async register(req, res) {
    const user = await User.create(req.body);
    Job.dispatch('SendWelcomeEmailJob', { userId: user.id });
    return redirect('/dashboard');
  }
}
```

`Job.dispatch()` is non blocking. it immediately returns control to the controller while the job executes in a background worker thread.

<a name="from-anywhere"></a>

### from anywhere

the `Job` facade is available globally. you can dispatch jobs from controllers, middleware, socket event handlers, other jobs, scheduled commands, or any file that imports it.

```javascript
import { Job } from 'dframework';

Job.dispatch('SyncInventoryJob', { warehouseId: 3 });
```

or simply use the global `Job` variable that is automatically available in all request contexts.

```javascript
Job.dispatch('GenerateReportJob', { month: 'june', year: 2026 });
```

<a name="from-the-command-line"></a>

### from the command line

you can also dispatch a job from the command line using the `dstrn dispatch` command. this is useful for testing jobs during development or running one off tasks manually.

```bash
dstrn dispatch SendWelcomeEmailJob --payload '{"userId": 1}'
```

the `--payload` flag accepts a json string that is parsed and passed to the job's `handle` method. if omitted, the job receives an empty object. the command bootstraps the full application context including database connectivity and all framework globals, exactly as a worker thread does. execution time is reported on completion.

```bash
dstrn dispatch CleanupExpiredSessionsJob
```

<a name="payload-data"></a>

### payload data

the second argument to `Job.dispatch()` is a plain javascript object that will be serialized to json and passed to the job's `handle` method. keep payloads small and serializable. pass identifiers rather than full objects.

```javascript
Job.dispatch('ProcessImageJob', {
  imageId: upload.id,
  operations: ['resize', 'compress'],
  quality: 85
});
```

<a name="the-job-facade"></a>

## the job facade

the `Job` global is a proxy facade that delegates to the application's `Queue` instance. it is automatically registered when the application boots and is available in the global scope.

```javascript
Job.dispatch('JobName', { key: 'value' });
```

if you attempt to use the `Job` facade before the application has initialized, it will throw a descriptive error.

<a name="how-the-worker-pool-works"></a>

## how the worker pool works

<a name="worker-isolation"></a>

### worker isolation

each dispatched job runs in its own dedicated `Worker` thread using node `worker_threads`. the worker thread initializes a minimal application instance that includes database connectivity, configuration loading, and all framework globals without starting an http server. this means your job code can use the exact same apis as your controllers including the `DB` facade, `Config`, model classes, and all helper functions.

<a name="concurrency-limits"></a>

### concurrency limits

the queue enforces a maximum number of concurrent workers. by default, up to four jobs can run simultaneously. if all worker slots are occupied when a new job is dispatched, the job is placed in an in memory queue and will be executed as soon as a worker becomes available.

the maximum worker count is configurable through the `queue.maxWorkers` configuration key.

<a name="automatic-queuing"></a>

### automatic queuing and persistent worker pool

when the worker pool is at capacity, additional dispatched jobs are automatically buffered in an internal queue. workers in dframework operate within a persistent worker thread pool:

1. lazy warming: the framework checks the application `jobs/` directory at startup. if no jobs exist, zero worker threads are spawned, ensuring applications with no background jobs incur zero memory or performance overhead.
2. persistent execution: when jobs exist, worker threads boot the application environment and database pool once and stay alive in the idle pool. subsequent jobs are picked up immediately by warm workers without thread creation or isolate boot delay.
3. automatic dequeuing: as each worker finishes its job, it picks up the next buffered job from the internal queue, or returns to the idle pool ready for future dispatches.
4. fault recovery: if an unhandled fatal error or process crash occurs inside a worker thread, the worker cleans up and exits, and a replacement worker is spawned to continue processing pending tasks.

<a name="job-lifecycle"></a>

## job lifecycle

<a name="bootstrapping"></a>

### bootstrapping

when a persistent worker thread boots, it performs the following steps:

1. loads environment variables and configuration files from the project
2. initializes an application instance with database connection and global facades
3. signals readiness to the main process and waits for dispatched jobs
4. upon receiving a job, imports the job class from the `jobs` directory
5. instantiates the job class and validates the `handle` method
6. invokes `handle(payload)` with the serialized payload
7. reports success or failure with execution duration back to the main thread
8. returns to the idle worker pool to immediately accept subsequent jobs

<a name="database-access"></a>

### database access

jobs have full access to the database through the same `DB` facade and model classes available in your controllers. each worker thread establishes its own database connection during initialization. the connection is guaranteed to be closed when the job completes through a cleanup function that runs in the worker's finally block, ensuring proper resource teardown regardless of whether the job succeeded or failed.

```javascript
export default class CleanupExpiredSessionsJob {
  async handle(payload) {
    const cutoff = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
    await DB.table('sessions').where('updated_at', '<', cutoff).delete();
  }
}
```

<a name="broadcasting-from-jobs"></a>

### broadcasting from jobs

jobs can dispatch real time websocket events and update component state on connected clients using the global `Socket` facade. because jobs run inside dedicated worker threads, `Socket.broadcast()` and `Socket.setState()` automatically relay messages over the internal worker channel to the main process, which broadcasts them to connected browsers.

```javascript
export default class GenerateReportJob {
  async handle(payload) {
    Socket.broadcast('report:progress', { percent: 25 });

    const report = await this.generate(payload.reportId);
    Socket.broadcast('report:progress', { percent: 100 });

    Socket.broadcast('report:ready', {
      reportId: report.id,
      downloadUrl: `/reports/${report.id}/download`
    });

    Socket.setState('#report-banner', {
      status: 'complete',
      reportId: report.id
    });
  }
}
```

no external message brokers (such as redis) are required. the worker thread communicates directly with the primary server instance.

<a name="error-handling"></a>

### error handling

if a job's `handle` method throws an error, a structured error message containing the error name, message, stack trace, and execution duration is sent back to the main thread and logged by the application logger. the worker thread runs its cleanup function to close database connections before exiting.

```
[JOB] [SendWelcomeEmailJob] failed after 1847ms:
Error: SMTP connection refused
    at Mailer.send (mailer/Mailer.js:42)
    at SendWelcomeEmailJob.handle (jobs/SendWelcomeEmailJob.js:12)
```

if the worker thread itself crashes (for example due to an out of memory condition), the main thread receives an `error` event and logs the crash. the worker slot is freed and the next queued job is processed normally. database connections are automatically released when the worker process terminates.

<a name="configuration"></a>

## configuration

the queue system reads its configuration from the `app.queue` config namespace.

| key                | type     | default | description                                 |
| ------------------ | -------- | ------- | ------------------------------------------- |
| `app.queue.maxWorkers` | `number` | `4`     | maximum number of concurrent worker threads |

```javascript
// config/app.js
export default {
  queue: {
    maxWorkers: Env.value('QUEUE_MAX_WORKERS', 4),
  },
};
```
