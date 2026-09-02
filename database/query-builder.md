# query builder

- [query builder](#query-builder)

  - [introduction](#introduction)

  - [methods overview](#methods-overview)

  - [retrieving results](#retrieving-results)

    - [getting all rows](#getting-all-rows)
    - [getting a single row](#getting-a-single-row)
    - [plucking values](#plucking-values)
    - [aggregates (count, sum, avg, min, max)](#aggregates)

  - [selects](#selects)
    - [subquery selects](#subquery-selects)

  - [where clauses](#where-clauses)

    - [basic where clauses](#basic-where-clauses)
    - [logical grouping (where closures)](#logical-grouping)
    - [or statements](#or-statements)
    - [additional where clauses](#additional-where-clauses)
    - [subqueries in where in](#subqueries-in-where-in)
    - [where exists and where not exists](#where-exists)
    - [column comparisons](#column-comparisons)

  - [conditional clauses (when and unless)](#conditional-clauses)

  - [joins](#joins)

    - [inner join](#inner-join)
    - [left join / right join](#left-join--right-join)
    - [advanced join clauses](#advanced-join-clauses)
    - [cross join](#cross-join)
    - [raw joins](#raw-joins)

  - [ordering, grouping, and limits](#ordering-grouping-and-limits)

  - [inserts](#inserts)

    - [bulk inserts](#bulk-inserts)

  - [updates](#updates)

  - [deletes](#deletes)

  - [auto hashing](#auto-hashing)

<a name="introduction"></a>

## introduction

dframework provides a fluent, chainable query builder that allows you to construct database queries. it protects against sql injection attacks by relying exclusively on prepared statements. you do not need to clean or sanitize bindings manually.

you begin a query builder chain by calling the `table` method on the globally available `DB` facade.

```javascript
const builder = DB.table('users');
// builder: TableQuery. chainable, thenable, and async iterable.
```

<a name="methods-overview"></a>

## methods overview

the query builder is built up by chaining constraint calls (which return the builder) and terminated by a terminal call (which returns a promise). the `returns` column below reflects the actual resolved value.

### chainable constraint methods

each of these returns the same `TableQuery` instance for chaining.

| method | arguments |
| --- | --- |
| `select(...columns)` | column names; replaces the current select list |
| `selectRaw(expression, bindings?)` | raw sql fragment plus optional bindings |
| `selectSub(query, as)` | scalar subquery expression with alias |
| `distinct(...columns?)` | mark distinct, optionally replacing the select list |
| `where(column, operator?, value?)` | column name/value, `{ column: value }` object, or closure `(q) => ...` |
| `whereNot(...)` | negated form of `where` (supports closure) |
| `orWhere(...)` | `or` form of `where` (supports closure) |
| `orWhereNot(...)` | `or` form of `whereNot` (supports closure) |
| `whereIn(column, values)` | column name plus `Array` of values or subquery closure `(q) => ...` |
| `whereNotIn(column, values)` | negated `whereIn` (supports subquery closure) |
| `orWhereIn(...)` / `orWhereNotIn(...)` | `or` forms of the IN clauses |
| `whereExists(callback)` / `whereNotExists(callback)` | exists / not exists subquery check |
| `orWhereExists(callback)` / `orWhereNotExists(callback)` | `or` forms of exists checks |
| `whereNull(column)` / `whereNotNull(column)` | null check |
| `whereBetween(column, [min, max])` / `whereNotBetween(...)` | range check, requires exactly two values |
| `whereColumn(column, operator?, otherColumn)` | compare two columns in the same row |
| `whereHashed(column, operator?, value)` / `orWhereHashed(...)` | compare against the fast hash of a value |
| `when(value, callback, defaultCallback?)` | apply callback if value is truthy |
| `unless(value, callback, defaultCallback?)` | apply callback if value is falsy |
| `join(table, first, operator?, second)` | inner join on another table (supports join closure) |
| `leftJoin(table, first, operator?, second)` | left outer join on another table (supports join closure) |
| `rightJoin(table, first, operator?, second)` | right outer join on another table (supports join closure) |
| `crossJoin(table)` | cross join on another table |
| `joinRaw(expression, bindings?)` | raw join clause with parameter bindings |
| `groupBy(...columns)` | grouping columns |
| `orderBy(column, direction='ASC')` | sort, direction is `'ASC'` or `'DESC'` |
| `limit(n)` / `offset(n)` | numeric row cap and skip count |
| `setHashFields(fields)` | override which columns are autohashed on this builder |

### terminal methods

these return a promise and execute the underlying query.

| method | arguments | returns |
| --- | --- | --- |
| `get()` | none | `Promise<Array<object>>` of matching rows; `[]` when none match |
| `first(where?)` | optional `{ column: value }` object | `Promise<object\|null>` (first matching row, or null) |
| `latest(count?, column?)` | `null` or number; optional column name | `null` count: `Promise<object\|null>`; numeric count: `Promise<Array<object>>` |
| `count(column?)` | optional column name (defaults to `*`) | `Promise<number>` (matching row count; group by returns the number of groups) |
| `sum(column)` | column name | `Promise<number>` (sum of values; 0 when no rows) |
| `avg(column)` | column name | `Promise<number\|null>` (average value; null when no rows) |
| `min(column)` | column name | `Promise<any>` (minimum value; null when no rows) |
| `max(column)` | column name | `Promise<any>` (maximum value; null when no rows) |
| `pluck(column)` | column name | `Promise<Array>` of that column's values across the matching rows (empty `[]` when none) |
| `getWithCount()` | none | `Promise<{ rows: Array<object>, total: number }>`; rows have the internal `_total_count` field stripped |
| `insert(data)` | row object, or array of row objects | single row: mysql2 `ResultSetHeader` (`insertId`, `affectedRows`); array of rows: `Array<number>` of generated ids (empty `[]` for an empty input array) |
| `update(data, where?)` | column/value object; optional where object | `Promise<object>` mysql2 `ResultSetHeader` |
| `save(data)` | alias for `update(data)` | `Promise<object>` mysql2 `ResultSetHeader` |
| `delete(where?)` | optional where object (or chained `where`) | `Promise<object>` mysql2 `ResultSetHeader`; throws when no where is set |

the builder itself is thenable and async iterable, so you can `await DB.table('users')` or `for await (const row of DB.table('users'))` directly. both forms execute the select and consume the same `Array<object>` shape.

<a name="retrieving-results"></a>

## retrieving results

<a name="getting-all-rows"></a>

### getting all rows

the `get` method executes the query and returns an array of result objects.

```javascript
const users = await DB.table('users').get();
// users: Array<object>. empty [] when no rows match.
```

the query builder is also an async iterable, allowing you to iterate directly over the builder instance without calling `get`.

```javascript
for await (const user of DB.table('users')) {
  console.log(user.name);
}
// yields plain row objects (not model instances).
```

<a name="getting-a-single-row"></a>

### getting a single row

if you only need to retrieve a single row, use the `first` method. it returns the object directly instead of wrapping it in an array.

```javascript
const user = await DB.table('users').where('email', 'tarou@example.com').first();
// user: object | null. null when no row matches.
```

<a name="latest-rows"></a>

### latest rows

the `latest` method orders rows by a column descending and returns either a single row or an array. it is shorthand for `orderBy(column, 'DESC').limit(n).get()`.

```javascript
const newest = await DB.table('posts').latest();
// newest: object | null
const recent = await DB.table('posts').latest(10);
// recent: Array<object>
const recentByPublished = await DB.table('posts').latest(10, 'published_at');
```

calling `latest()` with no arguments orders by `created_at` desc and returns the single most recent row (or null when none match). passing a numeric `count` returns that many rows as an array. the column defaults to `created_at` but may be overridden with the second argument.

<a name="plucking-values"></a>

### plucking values

if you want to retrieve a flat array containing the values of a single column, use the `pluck` method.

```javascript
const titles = await DB.table('posts').pluck('title');
// titles: Array. one entry per matching row, in the order returned by the database.
```

<a name="aggregates"></a>

### aggregates

the query builder provides helper methods for aggregating data: `count`, `sum`, `avg`, `min`, and `max`.

```javascript
const totalOrders = await DB.table('orders').where('status', 'pending').count();
// totalOrders: number. 0 when no rows match.

const totalRevenue = await DB.table('orders').where('status', 'completed').sum('amount');
// totalRevenue: number. 0 when no rows match.

const avgPrice = await DB.table('products').where('category_id', 4).avg('price');
// avgPrice: number | null. null when no rows match.

const cheapest = await DB.table('products').min('price');
const priciest = await DB.table('products').max('price');
```

all aggregates automatically respect joined tables, table aliases, and where constraints:

```javascript
const totalDuration = await DB.table('track_interactions')
  .join('tracks', 'track_interactions.track_id', '=', 'tracks.id')
  .where('track_interactions.user_id', 1)
  .where('track_interactions.type', 'like')
  .sum('tracks.duration');
```

<a name="selects"></a>

## selects

by default, the query builder selects all columns. to specify exact columns, use the `select` method.

```javascript
const users = await DB.table('users').select('id', 'name', 'email').get();
// users: Array<object>. each row only carries the requested columns.
```

if you need to insert a raw sql expression into the select clause, use the `selectRaw` method alongside any bindings.

```javascript
const users = await DB.table('users')
  .selectRaw('COUNT(id) as total, status')
  .groupBy('status')
  .get();
// users: Array<object>. each row has { total, status }.
```

to force the query to return only distinct results, use the `distinct` method. call it with column names to also set the select list.

```javascript
const activeRoles = await DB.table('users').distinct('role').get();
// activeRoles: Array<object>. each row has the distinct role values.
```

<a name="subquery-selects"></a>

### subquery selects

you can add subqueries into your select clause using the `selectSub` method. it accepts either a closure or a query builder instance, and an alias name.

```javascript
const users = await DB.table('users')
  .select('id', 'name')
  .selectSub(q => {
    q.from('orders').selectRaw('COUNT(*)').whereColumn('orders.user_id', 'users.id');
  }, 'orders_count')
  .get();
```

<a name="where-clauses"></a>

## where clauses

<a name="basic-where-clauses"></a>

### basic where clauses

the `where` method accepts three arguments: the column name, the operator, and the value. if you omit the operator, the builder assumes equality.

```javascript
await DB.table('users').where('votes', '=', 100).get();
await DB.table('users').where('votes', 100).get();
await DB.table('users').where('votes', '>=', 100).get();
await DB.table('users').where('name', 'LIKE', '%test%').get();
```

the `whereNot` method negates the condition.

```javascript
await DB.table('users').whereNot('status', 'inactive').get();
```

you can also pass an object to apply multiple equality conditions simultaneously.

```javascript
await DB.table('users').where({
  status: 'active',
  role: 'admin'
}).get();
```

<a name="logical-grouping"></a>

### logical grouping (where closures)

to create complex grouped boolean expressions (nested parenthesis), pass a closure to `where`, `whereNot`, `orWhere`, or `orWhereNot`. the closure receives a query builder instance to nest your constraints.

```javascript
const users = await DB.table('users')
  .where('active', 1)
  .where(q => {
    q.where('role', 'admin')
     .orWhere('role', 'superadmin');
  })
  .get();
// compiles to: WHERE `active` = ? AND (`role` = ? OR `role` = ?)
```

you can combine nested closures with `whereNot` or `orWhereNot` to wrap expressions in `NOT (...)`:

```javascript
const users = await DB.table('users')
  .where('tenant_id', 1)
  .whereNot(q => {
    q.where('status', 'banned')
     .orWhere('suspended', 1);
  })
  .get();
```

<a name="or-statements"></a>

### or statements

use the `orWhere` method to chain clauses with a logical or operator.

```javascript
await DB.table('users')
  .where('votes', '>', 100)
  .orWhere('name', 'tarou')
  .get();
```

the `orWhereNot` method is also available for negated or conditions.

<a name="additional-where-clauses"></a>

### additional where clauses

the query builder provides specialized methods for common condition types.

**whereIn / whereNotIn**
verifies that a given column's value is contained within an array, or matches a subquery closure.

```javascript
await DB.table('users').whereIn('id', [1, 2, 3]).get();
// when the array is empty, whereIn compiles to `0 = 1` (no rows)
// and whereNotIn compiles to `1 = 1` (all rows).
```

<a name="subqueries-in-where-in"></a>

### subqueries in where in

you can pass a closure callback to `whereIn` and `whereNotIn` to construct subquery filters:

```javascript
const customersWithOrders = await DB.table('users')
  .whereIn('id', q => {
    q.from('orders').select('user_id').where('status', 'paid');
  })
  .get();
// compiles to: WHERE `id` IN (SELECT `user_id` FROM `orders` WHERE `status` = ?)
```

**whereNull / whereNotNull**
verifies that the value of a column is or is not null.

```javascript
await DB.table('users').whereNull('deleted_at').get();
```

**whereBetween / whereNotBetween**
verifies that a column's value lies within two bounds. you must provide an array with exactly two values. both values must be defined (use `null` rather than `undefined` to keep a bound as sql null).

```javascript
await DB.table('users').whereBetween('votes', [1, 100]).get();
```

<a name="where-exists"></a>

### where exists and where not exists

use `whereExists` and `whereNotExists` (or their `orWhereExists` / `orWhereNotExists` counterparts) to write `EXISTS (SELECT ...)` subqueries:

```javascript
const users = await DB.table('users')
  .whereExists(q => {
    q.from('orders')
      .whereColumn('orders.user_id', 'users.id')
      .where('orders.total', '>', 100);
  })
  .get();
```

<a name="column-comparisons"></a>

### column comparisons

use the `whereColumn` method to compare the values of two different columns within the same row.

```javascript
await DB.table('users').whereColumn('updated_at', '>', 'created_at').get();
```

<a name="conditional-clauses"></a>

## conditional clauses (when and unless)

sometimes you want query clauses to apply only when a given condition is true. use `when` and `unless` to conditionally modify a query without breaking the method chain.

the `when` method executes the given closure if the first argument evaluates to truthy. an optional third closure executes if the condition is falsy.

```javascript
const role = req.query?.role;
const sortBy = req.query?.sort;

const users = await DB.table('users')
  .when(role, (q, val) => q.where('role', val))
  .when(sortBy, (q, val) => q.orderBy(val, 'ASC'), q => q.orderBy('id', 'DESC'))
  .get();
```

the `unless` method operates as the inverse: it executes the closure when the first argument is falsy.

```javascript
const users = await DB.table('users')
  .unless(includeInactive, q => q.where('active', 1))
  .get();
```

<a name="joins"></a>

## joins

the query builder supports joining multiple tables using inner joins, left/right outer joins, cross joins, and raw join expressions.

<a name="inner-join"></a>

### inner join

to perform a basic inner join, call the `join` method. you can specify the target table, the first column, an optional operator (defaults to `=`), and the second column.

```javascript
const users = await DB.table('users')
  .join('contacts', 'users.id', '=', 'contacts.user_id')
  .select('users.*', 'contacts.phone')
  .get();
```

when only three arguments are passed, the operator defaults to `=`:

```javascript
await DB.table('users')
  .join('contacts', 'users.id', 'contacts.user_id')
  .get();
```

<a name="left-join--right-join"></a>

### left join / right join

to perform a `LEFT JOIN` or `RIGHT JOIN`, use `leftJoin` or `rightJoin`:

```javascript
const tracks = await DB.table('tracks')
  .leftJoin('genres', 'tracks.genre_id', '=', 'genres.id')
  .select('tracks.*', 'genres.name as genre_name')
  .get();
```

you can chain multiple joins together to traverse relationships:

```javascript
const tracks = await DB.table('tracks')
  .leftJoin('track_artists', 'tracks.id', '=', 'track_artists.track_id')
  .leftJoin('artists', 'track_artists.artist_id', '=', 'artists.id')
  .select('tracks.*')
  .orderBy('artists.name', 'ASC')
  .get();
```

table aliases are supported in the table argument:

```javascript
await DB.table('tracks')
  .leftJoin('artists as a', 'tracks.artist_id', '=', 'a.id')
  .get();
```

<a name="advanced-join-clauses"></a>

### advanced join clauses

if you need to specify multiple join conditions or combine `ON` clauses with `WHERE` clauses on the joined table, pass a closure callback as the second argument to `join`, `leftJoin`, or `rightJoin`:

```javascript
const users = await DB.table('users')
  .join('contacts', j => {
    j.on('users.id', '=', 'contacts.user_id')
     .where('contacts.primary', '=', 1)
     .whereNull('contacts.deleted_at');
  })
  .get();
// compiles to: INNER JOIN `contacts` ON `users`.`id` = `contacts`.`user_id` AND `contacts`.`primary` = ? AND `contacts`.`deleted_at` IS NULL
```

join clauses support `on`, `orOn`, `where`, `orWhere`, `whereNull`, and `whereNotNull`.

<a name="cross-join"></a>

### cross join

to perform a cartesian product, use the `crossJoin` method:

```javascript
const combos = await DB.table('sizes')
  .crossJoin('colors')
  .get();
```

<a name="raw-joins"></a>

### raw joins

for complex join conditions or subqueries in joins, use `joinRaw`:

```javascript
await DB.table('users')
  .joinRaw('LEFT JOIN contacts ON contacts.user_id = users.id AND contacts.status = ?', ['active'])
  .get();
```

<a name="ordering-grouping-and-limits"></a>

## ordering, grouping, and limits

the `orderBy` method sorts the result set. the second argument specifies the direction, accepting either `ASC` or `DESC`.

```javascript
await DB.table('users')
  .orderBy('name', 'DESC')
  .get();
```

the `groupBy` method accepts one or more column names to group the results.

```javascript
await DB.table('users')
  .groupBy('account_id', 'status')
  .get();
```

the `limit` and `offset` methods restrict the number of records returned and specify the starting point.

```javascript
await DB.table('users')
  .offset(10)
  .limit(5)
  .get();
```

<a name="inserts"></a>

## inserts

the `insert` method accepts an object of column and value pairs to insert into the database.

```javascript
const result = await DB.table('users').insert({
  email: 'tarou@example.com',
  name: 'tarou'
});
// result: mysql2 ResultSetHeader (single row)
// use result.insertId to read the assigned auto increment id.
```

<a name="bulk-inserts"></a>

### bulk inserts

if you pass an array of objects to the `insert` method, the query builder will execute a single, highly optimized bulk insert statement and return an array of generated ids.

```javascript
const ids = await DB.table('users').insert([
  { email: 'tarou@example.com', name: 'tarou' },
  { email: 'satou@example.com', name: 'satou' }
]);
// ids: Array<number>. one generated id per inserted row, in order.
// pass an empty array to get [] back without executing any query.
```

<a name="updates"></a>

## updates

the `update` method updates existing records. it accepts an object containing the columns to modify and their new values. it affects any records matching the previously chained where clauses.

```javascript
const result = await DB.table('users')
  .where('id', 1)
  .update({ votes: 1 });
// result: mysql2 ResultSetHeader
// result.affectedRows tells you how many rows actually changed.
```

the `save` method acts as an alias for `update` and returns the same `ResultSetHeader`.

<a name="deletes"></a>

## deletes

the `delete` method removes records from the table.

> [!IMPORTANT]
> at least one where condition is required before executing `delete()`. calling `delete()` without constraints throws an exception to prevent accidental table truncation.

```javascript
const result = await DB.table('users')
  .where('status', 'inactive')
  .delete();
// result: mysql2 ResultSetHeader
// result.affectedRows tells you how many rows were removed.
```

<a name="auto-hashing"></a>

## auto hashing

the query builder is aware of sensitive columns and automatically hashes their values using bcrypt during inserts and updates. by default, any column named `password` or `secret` triggers this behavior.

you can override the default fields for a specific query using the `setHashFields` method.

```javascript
await DB.table('tokens')
  .setHashFields(['api_key'])
  .insert({ api_key: 'plain-text-key' });
// returns the single row insert result (mysql2 ResultSetHeader).
```

if you need to force hashing on an arbitrary value without relying on column names, the framework provides `hash` and `fastHash` wrappers exported from the `QueryBuilder` module.

```javascript
import { hash, fastHash } from 'dframework/QueryBuilder';

// uses slow, secure bcrypt (for passwords)
await DB.table('users').insert({
  custom_secret: hash('plain-text-password')
});

// uses fast, peppered sha256 (for indexable tokens)
await DB.table('tokens').insert({
  token_hash: fastHash('plain-text-token')
});
```

both wrappers return an opaque object (not a string). the query builder unpacks them at execution time, so never read their value directly.

you can also perform direct where comparisons against plaintext values if the column is configured as a hash field, using the `whereHashed` and `orWhereHashed` methods.

```javascript
const token = await DB.table('tokens').whereHashed('secret', 'plain-text-key').first();
// token: object | null. whereHashed hashes the value with sha256
// and compares it against the stored hash.
```
