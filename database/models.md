# models

- [models](#models)
  - [introduction](#introduction)
  - [defining models](#defining-models)
    - [table names](#table-names)
    - [primary keys](#primary-keys)
  - [retrieving models](#retrieving-models)
    - [magic finders](#magic-finders)
    - [pagination](#pagination)
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
```

you may use the `find` method to retrieve a specific record by its primary key.

```javascript
const user = await User.find(1);
```

<a name="magic-finders"></a>

### magic finders

the framework provides dynamic magic methods for retrieving records by a specific column. simply append the column name in camel case to the `findBy` prefix.

```javascript
const user = await User.findByEmail('test@example.com');
```

<a name="pagination"></a>

### pagination

to paginate records, use the `paginate` method. it automatically reads the `page` query string parameter from the current request context and constructs the limit and offset constraints.

```javascript
const results = await User.where('status', 'active').paginate(15);
```

the returned object contains the `data` array alongside pagination metadata like `total`, `current_page`, and `last_page`.

<a name="inserting-and-updating"></a>

## inserting and updating

to insert a new record, you can instantiate a new model instance, set attributes on it, and call the `save` method.

```javascript
const user = new User();
user.name = 'tarou';
user.email = 'tarou@example.com';
await user.save();
```

<a name="mass-assignment"></a>

### mass assignment

alternatively, you can use the static `create` method to insert a new record and retrieve the instantiated model in a single line. there is no mass assignment protection configuration required; the framework inherently trusts server side model interactions.

```javascript
const user = await User.create({
  name: 'tarou',
  email: 'tarou@example.com'
});
```

to update a model, you can either mutate its properties and call `save`, or use the `update` method directly.

```javascript
const user = await User.find(1);
await user.update({ status: 'active' });
```

<a name="first-or-create"></a>

### first or create

the `firstOrCreate` method will attempt to locate a record using the given column/value pairs. if the model can not be found, a record will be inserted with the attributes from the first argument, along with any optional attributes from the second argument.

```javascript
const user = await User.firstOrCreate(
  { email: 'tarou@example.com' },
  { name: 'tarou' }
);
```

the `updateOrCreate` method is also available.

<a name="deleting-models"></a>

## deleting models

to delete a model, call the `delete` method on a model instance.

```javascript
const user = await User.find(1);
await user.delete();
```

<a name="relationships"></a>

## relationships

models can define relationships to other models, allowing you to fluently traverse and query connected data.

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

once a relationship is defined, you can query it by calling the method, which returns a query builder.

```javascript
const activePosts = await user.posts().where('status', 'active').get();
```

<a name="eager-loading"></a>

### eager loading

when you access a relationship as a property, the framework will read the preloaded relation data. to prevent the n+1 query problem, use the `with` method to eager load relationships when fetching the parent models.

```javascript
const users = await User.with('profile', 'posts').limit(10).get();

for (const user of users) {
  // accessing user.posts does not trigger an additional query
  console.log(user.posts);
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
await user.load('posts', 'profile');
```

<a name="serialization"></a>

## serialization

when you cast a model to a json string or return it from a route, the framework automatically converts it using the `toJSON` method. this method serializes all attributes and eager loaded relationships.

<a name="hiding-attributes"></a>

### hiding attributes

you may wish to hide certain attributes, such as passwords or sensitive tokens, from the serialized output. define a static `hidden` array on your model to exclude these attributes.

```javascript
export default class User extends Model {
  static hidden = ['password', 'secret', 'api_key'];
}
```

by default, the framework hides `password`, `token`, `secret`, `api_key`, and `remember_token`.

<a name="mutations"></a>

## mutations

models dynamically proxy property accesses to their underlying attribute store. you interact with the model instance as if it were a plain javascript object.

if you need to manually encrypt a sensitive field that isn't handled by the query builder's auto hashing configuration, you can use the `hash` method on the model instance.

```javascript
const user = await User.find(1);
user.custom_secret = 'plain-text';
await user.hash('custom_secret');
await user.save();
```
