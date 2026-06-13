# database

- [introduction](#introduction)

- [configuration](#configuration)

- [executing raw queries](#executing-raw-queries)

  - [parameterized queries](#parameterized-queries)
  - [caching queries](#caching-queries)
  - [convenience methods](#convenience-methods)

- [sql helpers](#sql-helpers)

  - [raw expressions](#raw-expressions)
  - [escaping and sanitizing](#escaping-and-sanitizing)

<a name="introduction"></a>

## introduction

dframework provides a database abstraction layer built on top of mysql2. the core database service handles connection pooling, query caching, and parameter binding automatically. this document focuses on configuring the database, executing raw sql queries, and using the built in sql helpers. the fluent query builder and the active record implementation are covered in their own dedicated documentation sections.

<a name="configuration"></a>

## configuration

database configuration is managed through environment variables and resolved via the `Config` facade under the `app.database` namespace. the framework expects the `host`, `user`, `pass`, and `name` values to be defined.

the connection pool is configured automatically when the application boots. by default, the framework maintains a pool of up to ten connections, utilizing connection keep alive and strict idle timeout management to prevent memory leaks and dropped connections. connection pooling guarantees that your application scales gracefully without exhausting database server limits.

<a name="executing-raw-queries"></a>

## executing raw queries

the database service is available globally throughout your application using the `DB` facade. no imports are necessary. you execute raw sql statements using the `query` method.

<a name="parameterized-queries"></a>

### parameterized queries

to prevent sql injection vulnerabilities, you must always use parameterized queries. pass an array of bindings as the second argument to the `query` method. the framework relies on prepared statements behind the scenes.

```javascript
const rows = await DB.query('SELECT * FROM users WHERE status = ? AND age > ?', ['active', 18]);
```

the `query` method always returns an array of results for select statements. for insert, update, or delete statements, it returns an object containing the affected rows and the last insert id.

<a name="caching-queries"></a>

### caching queries

dframework includes a memory bound query cache. you enable caching for a specific select query by passing a configuration object as the third argument.

```javascript
const rows = await DB.query(
  'SELECT * FROM settings WHERE scope = ?',
  ['global'],
  { cache: true, ttl: 1000 }
);
```

the `ttl` parameter defines the cache lifespan in milliseconds. the default is five hundred milliseconds.

the caching system is intelligent. the framework parses your raw sql to extract the table names involved. if you execute an `INSERT`, `UPDATE`, or `DELETE` statement against a table, the framework automatically flushes any cached select queries that reference that table. this guarantees that subsequent reads will instantly reflect the mutation without requiring manual cache invalidation.

<a name="convenience-methods"></a>

### convenience methods

for simple operations, the database facade provides shorthand methods that generate the raw sql for you.

the `first` method returns the first matching row or null.

```javascript
const user = await DB.first('users', { email: 'tarou@example.com' }, ['id', 'name']);
```

the `insert` method accepts a table name and a data object, returning the insert result.

```javascript
const result = await DB.insert('users', { name: 'tarou', email: 'tarou@example.com' });
console.log(result.insertId);
```

the `update` method accepts a table name, a data object containing the changes, and a where clause object.

```javascript
await DB.update('users', { status: 'active' }, { id: 12 });
```

<a name="sql-helpers"></a>

## sql helpers

dframework includes a dedicated module for formatting and securing sql expressions, accessible via the `SqlHelpers` export.

<a name="raw-expressions"></a>

### raw expressions

there are times when you need to bypass standard parameter escaping to execute a database function. you achieve this using the `raw` helper.

```javascript
import { SqlHelpers } from 'dframework';

await DB.update('users', { updated_at: SqlHelpers.raw('NOW()') }, { id: 1 });
```

to maintain strict security and prevent injection, the `raw` helper operates against a strict allowlist. it currently only accepts the following static functions: `CURRENT_TIMESTAMP`, `CURRENT_DATE`, `CURRENT_TIME`, and `NOW()`. passing any other string, especially one containing user input, will instantly throw an exception and halt execution.

<a name="escaping-and-sanitizing"></a>

### escaping and sanitizing

the `escapeIdentifier` method wraps table or column names in backticks, escaping any existing backticks safely.

```javascript
import { SqlHelpers } from 'dframework';

const column = SqlHelpers.escapeIdentifier('user`name'); // returns `user``name`
```

the `sanitizeColumn` method strips all characters except alphanumeric characters and underscores from a column name. it also supports standard dot notation for joined tables.

```javascript
const safe = SqlHelpers.sanitizeColumn('users.first-name;'); // returns users.firstname
```

the `sqlValue` method formats any javascript variable into a string safe for direct insertion into a raw sql query. it escapes strings, converts booleans to integers, handles nulls, and unpacks expressions generated by the `raw` helper.
