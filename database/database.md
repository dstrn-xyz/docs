# database

- [database](#database)
  - [introduction](#introduction)
  - [configuration](#configuration)
  - [database facade](#database-facade)

    - [methods overview](#methods-overview)
  - [executing raw queries](#executing-raw-queries)

    - [parameterized queries](#parameterized-queries)
    - [caching queries](#caching-queries)
    - [return contract](#return-contract)
    - [transparent query batching](#transparent-query-batching)
  - [sql helpers](#sql-helpers)

    - [raw expressions](#raw-expressions)
    - [escaping and sanitizing](#escaping-and-sanitizing)

<a name="introduction"></a>

## introduction

dframework provides a unified database abstraction layer supporting mysql, postgresql, and sqlite. the core database service handles connection pooling, query caching, transparent select batching, dialect compilation, savepoint transactions, and parameter binding automatically. this document covers configuring the database, executing raw sql queries, and using the built in sql helpers. the fluent query builder and the active record implementation are covered in their own dedicated documentation sections.

<a name="configuration"></a>

## configuration

database configuration is managed through environment variables and resolved via the `Config` facade under the `database` namespace in `config/database.js`. the framework supports `driver` (`mysql`, `pgsql`, `sqlite`), `host`, `port`, `user`, `pass`, `name`, and `connectionLimit` values.

| option            | environment variable  | default                             | description                                      |
| ----------------- | --------------------- | ----------------------------------- | ------------------------------------------------ |
| `driver`          | `DB_DRIVER`           | `mysql`                             | database driver (`mysql`, `pgsql`, `sqlite`)     |
| `host`            | `DB_HOST`             | `localhost`                         | database server hostname                         |
| `port`            | `DB_PORT`             | `3306` (mysql) / `5432` (pgsql)     | database server port                             |
| `user`            | `DB_USER`             | `root` (mysql) / `postgres` (pgsql) | database user                                    |
| `pass`            | `DB_PASS`             | `""`                                | database password                                |
| `name`            | `DB_NAME`             | `dframework`                        | database name (or sqlite file path / `:memory:`) |
| `connectionLimit` | `DB_CONNECTION_LIMIT` | `10` (production `20`)              | maximum connection pool size                     |
| `maxIdle`         | `DB_MAX_IDLE`         | `5`                                 | maximum idle connections                         |
| `idleTimeout`     | `DB_IDLE_TIMEOUT`     | `60000`                             | idle connection timeout in milliseconds          |

for mysql, connection pooling uses `mysql2/promise` (requires `npm install mysql2`) with keep alive and decimal numbers enabled. for postgresql, connection pooling uses `pg` (requires `npm install pg`), compiling positional parameter bindings (`$1`, `$2`) and returning clauses automatically. for sqlite, the framework uses built in `node:sqlite` (available in node 22.5.0 or higher), enabling write ahead logging (`WAL`) and busy timeouts automatically with zero external dependencies.

the pool or database connection is created lazily on the first query (no upfront connection is established during boot), so a misconfigured database does not prevent the process from starting.

<a name="database-facade"></a>

## database facade

the `DB` facade is a proxy over the active app's pooled database instance. it resolves the request bound `app.db`, or the global `app.db` when called outside a request context. calling `DB` before the app has booted throws an explicit diagnostic error rather than silently returning undefined.

<a name="methods-overview"></a>

### methods overview

the `DB` facade exposes the following methods. the `returns` column reflects the actual resolved value, including empty array and null semantics.

| method                             | arguments                                                         | returns                                                                                                                                                                     |
| ---------------------------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `DB.query(sql, params?, options?)` | raw parameterized sql; params array; `{ cache, ttl, _skipBatch }` | `Array<object>` for select statements (empty `[]` when no rows), or normalized mutation header for insert/update/delete (carries `affectedRows`, `changedRows`, `insertId`) |
| `DB.table(name)`                   | table name string                                                 | a `TableQuery` query builder bound to this table, chainable; never null                                                                                                     |
| `DB.insert(table, data)`           | table name; column/value object                                   | normalized mutation header (use `result.insertId` or `result.affectedRows`)                                                                                                 |
| `DB.transaction(fn)`               | `async (conn) => result`                                          | whatever the `fn` callback resolves to; supports nested savepoints (`sp_1`, `sp_2`) and rethrows after rollback                                                             |
| `DB.databaseExists()`              | none                                                              | `Promise<boolean>`                                                                                                                                                          |
| `DB.ensureDatabaseExists()`        | none                                                              | `Promise<void>`; throws a diagnostic error when `database.name` is missing                                                                                                  |
| `DB.ensureTableExists(table)`      | table name                                                        | `Promise<void>`; throws when the table is absent (used internally by the query builder)                                                                                     |
| `DB.close()`                       | none                                                              | `Promise<void>`; ends the pool and clears the shared query cache                                                                                                            |

the `pool` getter returns the underlying driver pool or sqlite database instance when you need direct engine access. most code never needs it.

<a name="executing-raw-queries"></a>

## executing raw queries

you execute raw sql statements using the `query` method. the `DB` facade is available globally throughout your application, no imports needed.

<a name="parameterized-queries"></a>

### parameterized queries

always use parameterized queries to prevent sql injection. pass an array of bindings as the second argument. the framework relies on prepared statements behind the scenes.

```javascript
const rows = await DB.query(
  'SELECT * FROM users WHERE status = ? AND age > ?',
  ['active', 18]
);
// rows: Array<object>, one entry per matching row. [] when no rows match.
```

<a name="caching-queries"></a>

### caching queries

dframework includes a memory bound query cache. enable it for a specific select by passing a configuration object as the third argument.

```javascript
const rows = await DB.query(
  'SELECT * FROM settings WHERE scope = ?',
  ['global'],
  { cache: true, ttl: 1000 }
);
// rows: Array<object>. identical to the uncached return shape.
```

the `ttl` parameter defines the cache lifespan in milliseconds. the default is five hundred milliseconds.

the caching system parses your raw sql to extract referenced table names. when you execute an `INSERT`, `UPDATE`, or `DELETE` against a table, the framework automatically flushes any cached select queries that reference that table. this keeps subsequent reads consistent without manual invalidation.

the cache is LRU and capped at `database.queryCache.maxEntries` entries per request (default `200`). promote hot queries to most recently used simply by reading them; you do not need to evict cold entries manually.

<a name="return-contract"></a>

### return contract

the `query` method's resolved value depends on the statement type. this is the single most important contract to internalize.

| statement                                             | returns                                      | empty / no match                                            |
| ----------------------------------------------------- | -------------------------------------------- | ----------------------------------------------------------- |
| `SELECT` / `SHOW` / `DESCRIBE` / `EXPLAIN` / `PRAGMA` | `Array<object>`                              | `[]`                                                        |
| `WITH ... SELECT` (no mutation)                       | `Array<object>`                              | `[]`                                                        |
| `INSERT`                                              | `{ affectedRows, changedRows, insertId }`    | carries `insertId`, `affectedRows: 0` when nothing inserted |
| `UPDATE`                                              | `{ affectedRows, changedRows, insertId: 0 }` | `affectedRows: 0` when nothing matched                      |
| `DELETE`                                              | `{ affectedRows, changedRows, insertId: 0 }` | `affectedRows: 0` when nothing matched                      |

the framework normalizes query execution results across mysql, postgresql, and sqlite into a unified contract. you always receive the rows array (for selects) or a normalized mutation header `{ affectedRows, changedRows, insertId }` (for mutations). the mutation header is a plain object; it has no `length`, so guard with `affectedRows` rather than array checks.

<a name="transparent-query-batching"></a>

### transparent query batching

selects go through a per request batcher that coalesces simple equality and IN lookups (`SELECT * FROM table WHERE column = ?` and `SELECT * FROM table WHERE column IN (?, ?, ?)`) issued within the same microtask into a single `IN (?, ?, ...)` query. identical parameter values are deduplicated.

this is fully transparent: callers receive the rows that match their specific value, as if they had issued their own query. the batcher only kicks in for the patterns above; anything more complex (joins, `AND status = ?` mixed in, aliased projections, `null`/`undefined` params) bypasses batching and runs the original query directly.

to intentionally skip batching (for example inside a batcher's own callback or for cursor sensitive logic), pass `{ _skipBatch: true }` in the options argument of `DB.query`.

<a name="sql-helpers"></a>

## sql helpers

dframework includes a dedicated module for formatting and securing sql expressions, accessible via the `SqlHelpers` export.

<a name="raw-expressions"></a>

### raw expressions

there are times when you need to bypass standard parameter escaping to execute a database function. you achieve this using the `raw` helper.

```javascript
import { SqlHelpers } from 'dframework';

await DB.table('users').update({ updated_at: SqlHelpers.raw('NOW()') }, { id: 1 });
```

> [!IMPORTANT]
> to prevent sql injection, `SqlHelpers.raw()` operates against an allowlist accepting only `CURRENT_TIMESTAMP`, `CURRENT_DATE`, `CURRENT_TIME`, `NOW()`, `UUID()`, `UUID_SHORT()`, `UNIX_TIMESTAMP()`, `UTC_TIMESTAMP()`, `UTC_DATE()`, `UTC_TIME()`, `RAND()`, `NULL`, `TRUE`, `FALSE`, and `DEFAULT`. passing any other expression throws an immediate diagnostic error.

| helper                                               | arguments                              | returns                                                                                                                      |
| ---------------------------------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| `SqlHelpers.raw(expr)`                               | allowlisted sql function string        | `{ [Symbol.for('dframework.sqlRaw')]: true, value: string }` wrapper; consumed by the query builder to inline the expression |
| `SqlHelpers.escapeIdentifier(id)`                    | identifier string                      | backtick wrapped string with embedded backticks doubled                                                                      |
| `SqlHelpers.escapeTable(table, wrap?)`               | table name or table as alias           | escaped table expression                                                                                                     |
| `SqlHelpers.escapeColumn(col, wrap?)`                | column name or table.column            | escaped column expression                                                                                                    |
| `SqlHelpers.validateIdentifier(id, name?)`           | identifier string, descriptor string   | validated identifier string (throws on illegal characters)                                                                   |
| `SqlHelpers.sanitizeTable(name)`                     | table name                             | sanitized table name string                                                                                                  |
| `SqlHelpers.sanitizeColumn(col)`                     | column name, optionally `table.column` | input with characters outside `[a-zA-Z0-9_]` stripped                                                                        |
| `SqlHelpers.sqlValue(val, booleanKeyword?, isAnsi?)` | any js value                           | safe sql literal string                                                                                                      |

<a name="escaping-and-sanitizing"></a>

### escaping and sanitizing

the `escapeIdentifier` method wraps table or column names in backticks, escaping any existing backticks safely.

```javascript
import { SqlHelpers } from 'dframework';

const column = SqlHelpers.escapeIdentifier('user`name'); // returns `user``name`
```

the `sanitizeColumn` method strips all characters except alphanumeric characters and underscores from a column name. it also supports standard dot notation for joined tables.

```javascript
const safe = SqlHelpers.sanitizeColumn('users.first_name;'); // returns users.first_name
```

the `sqlValue` method formats any javascript variable into a string safe for direct insertion into a raw sql query. it escapes strings, converts booleans to integers or keywords, handles nulls, and unpacks expressions generated by the `raw` helper.