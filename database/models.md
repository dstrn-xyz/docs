# models

- [models](#models)
  - [introduction](#introduction)
  - [defining models](#defining-models)

    - [table names](#table-names)
    - [primary keys](#primary-keys)
  - [retrieving models](#retrieving-models)

    - [methods overview](#methods-overview)
    - [magic finders](#magic-finders)
    - [pagination](#pagination)
    - [latest rows](#latest-rows)
  - [inserting and updating](#inserting-and-updating)

    - [mass assignment](#mass-assignment)
    - [first or create](#first-or-create)
  - [deleting models](#deleting-models)
  - [relationships](#relationships)

    - [one to one](#one-to-one)
    - [one to many](#one-to-many)
    - [belongs to](#belongs-to)
    - [eager loading](#eager-loading)
    - [lazy eager loading](#lazy-eager-loading)
  - [serialization](#serialization)

    - [hiding attributes](#hiding-attributes)
  - [mutations](#mutations)

<a name="introduction"></a>

## introduction

dframework includes an active record implementation for interacting with your database. each database table has a corresponding model that is used to interact with that table. models allow you to query for data in your tables, as well as insert new records into the table, using an elegant and fluent interface.

<a name="defining-models"></a>

## defining models

to create a model, simply extend the base `Model` class provided by the framework.

```javascript
import { Model } from 'dframework';

export default class User extends Model {
  // 
}
```

<a name="table-names"></a>

### table names

by default, the framework will use the lowercased, plural name of the class as the table name. for example, the `User` model will assume a `users` table exists. if your table does not follow this convention, you may specify a custom table name by overriding the static `table` property.

```javascript
export default class User extends Model {
  static table = 'system_users';
}
```

<a name="primary-keys"></a>

### primary keys

the framework will automatically determine your table's primary key by inspecting the database schema and caching the result. if you want to override this behavior, you can define a static `primaryKey` property.

```javascript
export default class User extends Model {
  static primaryKey = 'uuid';
}
```

composite primary keys are automatically supported if they are defined in the schema.

<a name="retrieving-models"></a>

## retrieving models

models proxy all methods from the fluent query builder, allowing you to chain constraints before fetching the results. the `all` method will retrieve all of the records from the model's table.

```javascript
const users = await User.all();
// users: Array<User>. hydrated model instances. [] when the table is empty.
```

you may use the `find` method to retrieve a specific record by its primary key.

```javascript
const user = await User.find(1);
// user: User | null. null when no row matches the given id.
```

<a name="methods-overview"></a>

### methods overview

every method that runs a query returns a `Promise`. the result column lists what that promise resolves with, including null and empty array semantics.

| method | arguments | returns |
| --- | --- | --- |
| `Model.all()` | none | `Promise<Model[]>` (hydrated instances, `[]` when no rows) |
| `Model.find(id)` | primary key value, or `{ pk1, pk2 }` for composite keys | `Promise<Model\|null>` |
| `Model.first(where?)` | optional `{ column: value }` object | `Promise<Model\|null>` |
| `Model.latest(count?, column?)` | `null` or number; optional column name | `null` count: `Promise<Model\|null>`; numeric count: `Promise<Model[]>` |
| `Model.where(...)` | column/operator/value, or object | `ModelQueryBuilder` (chainable, thenable, async iterable) |
| `Model.orderBy(...)` / `groupBy(...)` / `limit(...)` / `offset(...)` / `distinct(...)` | chain values | `ModelQueryBuilder` (chainable) |
| `Model.with(...relations)` | dot notation relation names | `ModelQueryBuilder` (chainable) |
| `Model.paginate(perPage, pageName?)` | rows per page (default 10), optional page param name | `Promise<Paginator>` |
| `Model.firstOrCreate(attributes, values?)` | attributes object; optional extra values | `Promise<Model>` (existing or newly created) |
| `Model.updateOrCreate(attributes, values?)` | attributes object; optional extra values | `Promise<Model>` (updated or newly created) |
| `Model.create(data)` | column/value object | `Promise<Model>` (reloaded from the database using the primary key) |
| `instance.save(newData?)` | optional object to merge before saving | `Promise<void>` |
| `instance.update(data)` | column/value object | `Promise<void>` |
| `instance.delete()` | none | `Promise<void>` |
| `instance.hash(field)` | attribute name on the instance | `Promise<void>` (mutates the instance in place) |
| `instance.load(...relations)` | relation names | `Promise<Model>` (the same instance, with relations populated) |
| `instance.toJSON()` | none | plain object with hidden keys removed and relations serialized |

the `where` chainable builder exposes the same constraint, closure grouping, conditional (`when`, `unless`), subquery (`whereExists`, `whereIn`, `selectSub`), and aggregate methods as the fluent query builder (`where`, `whereIn`, `whereNull`, `whereBetween`, `whereColumn`, `whereHashed`, `whereExists`, `when`, `unless`, `selectSub`, `join`, `leftJoin`, `rightJoin`, `crossJoin`, `joinRaw`, `count`, `sum`, `avg`, `min`, `max`, etc.). it is also thenable (`await builder`) and async iterable (`for await (const m of builder)`).

<a name="magic-finders"></a>

### magic finders

the framework provides dynamic magic methods for retrieving records by a specific column. simply append the column name in camel case to the `findBy` prefix.

```javascript
const user = await User.findByEmail('test@example.com');
// user: User | null. resolves via findBy => where({ email }).first()
```

<a name="pagination"></a>

### pagination

to paginate records, use the `paginate` method. it automatically reads the active page query string parameter from the current request context, constructs limit and offset constraints, and returns a `Paginator` instance.

```javascript
const results = await User.where('status', 'active').paginate(15);
// results is a Paginator instance
// directly iterable: for (const user of results) or @foreach(results as user)
// access metadata: results.total, results.currentPage, results.lastPage
// render html: results.links() or @pagination(results)
```

you can also specify a custom page query parameter name as the second argument:

```javascript
const users = await User.paginate(15, 'user_page');
```

<a name="latest-rows"></a>

### latest rows

the `latest` method orders rows by a column descending and returns either a single model or an array.

```javascript
const newestPost = await Post.latest();
const recentPosts = await Post.latest(10);
const recentlyPublished = await Post.latest(10, 'published_at');
```

calling `latest()` with no arguments returns the single most recent instance (or null). passing a numeric `count` returns that many model instances as an array. the column defaults to `created_at` but may be overridden with the second argument.

<a name="inserting-and-updating"></a>

## inserting and updating

to insert a new record, you can instantiate a new model instance, set attributes on it, and call the `save` method.

```javascript
const user = new User();
user.name = 'tarou';
user.email = 'tarou@example.com';
await user.save();
// returns undefined. the instance now carries the assigned primary key
// (if the table has an auto increment column).
```

<a name="mass-assignment"></a>

### mass assignment

alternatively, you can use the static `create` method to insert a new record and retrieve the instantiated model in a single line. there is no mass assignment protection configuration required; the framework inherently trusts server side model interactions.

```javascript
const user = await User.create({
  name: 'tarou',
  email: 'tarou@example.com'
});
// user: User. reloaded from the database after insert, so it carries
// the generated primary key and any column defaults.
```

to update a model, you can either mutate its properties and call `save`, or use the `update` method directly.

```javascript
const user = await User.find(1);
await user.update({ status: 'active' });
// returns undefined. the instance attributes are mutated in place
// before the underlying UPDATE executes.
```

<a name="first-or-create"></a>

### first or create

the `firstOrCreate` method will attempt to locate a record using the given column/value pairs. if the model can not be found, a record will be inserted with the attributes from the first argument, along with any optional attributes from the second argument.

```javascript
const user = await User.firstOrCreate(
  { email: 'tarou@example.com' },
  { name: 'tarou' }
);
// user: User. always a hydrated instance; either the matched row
// or the one created by inserting { ...attributes, ...values }.
```

the `updateOrCreate` method is also available. it returns the existing instance with the update applied, or a new instance if none matched.

<a name="deleting-models"></a>

## deleting models

to delete a model, call the `delete` method on a model instance.

```javascript
const user = await User.find(1);
await user.delete();
// returns undefined. throws a diagnostic error when the instance
// is missing the primary key value(s) required to scope the delete.
```

<a name="relationships"></a>

## relationships

models can define relationships to other models, allowing you to fluently traverse and query connected data. every relationship method returns a `ModelQueryBuilder` (chainable, thenable, and awaitable).

<a name="one-to-one"></a>

### one to one

a one to one relationship is defined using the `hasOne` method. it requires the related model class and the foreign key name.

```javascript
import Profile from './Profile.js';

export default class User extends Model {
  profile() {
    return this.hasOne(Profile, 'user_id');
  }
}
```

<a name="one-to-many"></a>

### one to many

a one to many relationship is defined using the `hasMany` method.

```javascript
import Post from './Post.js';

export default class User extends Model {
  posts() {
    return this.hasMany(Post, 'user_id');
  }
}
```

<a name="belongs-to"></a>

### belongs to

the inverse of a `hasOne` or `hasMany` relationship is defined using the `belongsTo` method.

```javascript
import User from './User.js';

export default class Post extends Model {
  user() {
    return this.belongsTo(User, 'user_id');
  }
}
```

once a relationship is defined, you can query it by calling the method, which returns a `ModelQueryBuilder`.

```javascript
const activePosts = await user.posts().where('status', 'active').get();
// activePosts: Array<Post>. empty [] when no posts match.
```

awaiting a `hasMany`/`belongsTo`/`hasOne` relation builder returns the hydrated model(s) directly (array for `hasMany`, single model or null for `hasOne`/`belongsTo`), so you can also write:

```javascript
const profile = await user.profile(); // Profile | null
const posts = await user.posts();      // Array<Post>
```

you can also await the property directly without calling it as a function:

```javascript
const posts = await user.posts;
```

### automatic in memory caching and lazy fetching

the framework automatically caches eager loaded relations and handles lazy loading transparently when awaited:

- if the relation was eager loaded with `with('posts')` or `load('posts')`, `await user.posts` (or `await user.posts()`) resolves immediately in memory with zero database queries.
- if the relation was not eager loaded, `await user.posts` executes a single database query on demand.

because relation properties are callable proxies (enabling both in memory collection access and fluent query chaining like `user.posts().where(...)`), `Array.isArray(user.posts)` will return `false`. you do not need manual `Array.isArray` branches to optimize data access; simply `await user.posts` to get the hydrated collection whether it was preloaded or not.

<a name="eager-loading"></a>

### eager loading

when you access a relationship as a property, the framework will read the preloaded relation data. to prevent the n+1 query problem, use the `with` method to eager load relationships when fetching the parent models.

```javascript
const users = await User.with('profile', 'posts').limit(10).get();
// users: Array<User>. each user already has _relations populated.

for (const user of users) {
  // accessing user.posts does not trigger an additional query
  console.log(user.posts); // Array<Post>
}
```

you can eager load nested relationships using dot notation.

```javascript
const users = await User.with('posts.comments').get();
```

<a name="lazy-eager-loading"></a>

### lazy eager loading

if you have already retrieved a model instance and need to eager load a relationship after the fact, use the `load` method.

```javascript
const user = await User.find(1);
const sameUser = await user.load('posts', 'profile');
// returns the same instance with the relations populated.
```

<a name="serialization"></a>

## serialization

when you cast a model to a json string or return it from a route, the framework automatically converts it using the `toJSON` method. this method returns a plain object with all attributes and eager loaded relationships, applying the `hidden` list.

<a name="hiding-attributes"></a>

### hiding attributes

you may wish to hide certain attributes, such as passwords or sensitive tokens, from the serialized output. define a static `hidden` array on your model to exclude these attributes.

```javascript
export default class User extends Model {
  static hidden = ['password', 'secret', 'api_key'];
}
```

> [!NOTE]
> by default, the framework automatically hides `password`, `token`, `secret`, `api_key`, and `remember_token` when serializing models with `toJSON()`.

<a name="mutations"></a>

## mutations

models dynamically proxy property accesses to their underlying attribute store. you interact with the model instance as if it were a plain javascript object.

if you need to manually encrypt a sensitive field that isn't handled by the query builder's auto hashing configuration, you can use the `hash` method on the model instance.

```javascript
const user = await User.find(1);
user.custom_secret = 'plain-text';
await user.hash('custom_secret');
await user.save();
// hash() mutates the instance in place; save() persists it.
```
