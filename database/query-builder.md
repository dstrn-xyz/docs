# query builder

- [query builder](#query-builder)

  - [introduction](#introduction)

  - [retrieving results](#retrieving-results)

    - [getting all rows](#getting-all-rows)
    - [getting a single row](#getting-a-single-row)
    - [plucking values](#plucking-values)
    - [counting results](#counting-results)

  - [selects](#selects)

  - [where clauses](#where-clauses)

    - [basic where clauses](#basic-where-clauses)
    - [or statements](#or-statements)
    - [additional where clauses](#additional-where-clauses)
    - [column comparisons](#column-comparisons)

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

<a name="retrieving-results"></a>

## retrieving results

<a name="getting-all-rows"></a>

### getting all rows

the `get` method executes the query and returns an array of result objects.

```javascript
const users = await DB.table('users').get();
```

the query builder is also an async iterable, allowing you to iterate directly over the builder instance without calling `get`.

```javascript
for await (const user of DB.table('users')) {
  console.log(user.name);
}
```

<a name="getting-a-single-row"></a>

### getting a single row

if you only need to retrieve a single row, use the `first` method. it returns the object directly instead of wrapping it in an array.

```javascript
const user = await DB.table('users').where('email', 'tarou@example.com').first();
```

<a name="plucking-values"></a>

### plucking values

if you want to retrieve a flat array containing the values of a single column, use the `pluck` method.

```javascript
const titles = await DB.table('posts').pluck('title');
```

<a name="counting-results"></a>

### counting results

to determine the total number of records matching your criteria, use the `count` method. it executes a raw count aggregation and returns an integer.

```javascript
const total = await DB.table('orders').where('status', 'pending').count();
```

<a name="selects"></a>

## selects

by default, the query builder selects all columns. to specify exact columns, use the `select` method.

```javascript
const users = await DB.table('users').select('id', 'name', 'email').get();
```

if you need to insert a raw sql expression into the select clause, use the `selectRaw` method alongside any bindings.

```javascript
const users = await DB.table('users')
  .selectRaw('COUNT(id) as total, status')
  .groupBy('status')
  .get();
```

to force the query to return only distinct results, use the `distinct` method.

```javascript
const activeRoles = await DB.table('users').distinct('role').get();
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
verifies that a given column's value is contained within an array.

```javascript
await DB.table('users').whereIn('id', [1, 2, 3]).get();
```

**whereNull / whereNotNull**
verifies that the value of a column is or is not null.

```javascript
await DB.table('users').whereNull('deleted_at').get();
```

**whereBetween / whereNotBetween**
verifies that a column's value lies within two bounds. you must provide an array with exactly two values.

```javascript
await DB.table('users').whereBetween('votes', [1, 100]).get();
```

<a name="column-comparisons"></a>

### column comparisons

use the `whereColumn` method to compare the values of two different columns within the same row.

```javascript
await DB.table('users').whereColumn('updated_at', '>', 'created_at').get();
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

the `insert` method accepts an object of column and value pairs to insert into the database. it returns the array of created ids or a result object containing the `insertId` property.

```javascript
const result = await DB.table('users').insert({
  email: 'tarou@example.com',
  name: 'tarou'
});
```

<a name="bulk-inserts"></a>

### bulk inserts

if you pass an array of objects to the `insert` method, the query builder will execute a single, highly optimized bulk insert statement.

```javascript
await DB.table('users').insert([
  { email: 'tarou@example.com', name: 'tarou' },
  { email: 'satou@example.com', name: 'satou' }
]);
```

<a name="updates"></a>

## updates

the `update` method updates existing records. it accepts an object containing the columns to modify and their new values. it affects any records matching the previously chained where clauses.

```javascript
await DB.table('users')
  .where('id', 1)
  .update({ votes: 1 });
```

the `save` method acts as an alias for `update`.

<a name="deletes"></a>

## deletes

the `delete` method removes records from the table. for safety, the framework requires at least one where clause to be present before executing a delete. calling `delete` without conditions will throw an exception to prevent accidental table truncation.

```javascript
await DB.table('users')
  .where('status', 'inactive')
  .delete();
```

<a name="auto-hashing"></a>

## auto hashing

the query builder is aware of sensitive columns and automatically hashes their values using bcrypt during inserts and updates. by default, any column named `password` or `secret` triggers this behavior.

you can override the default fields for a specific query using the `setHashFields` method.

```javascript
await DB.table('tokens')
  .setHashFields(['api_key'])
  .insert({ api_key: 'plain-text-key' });
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

you can also perform direct where comparisons against plaintext values if the column is configured as a hash field, using the `whereHashed` and `orWhereHashed` methods.

```javascript
await DB.table('tokens').whereHashed('secret', 'plain-text-key').first();
```
