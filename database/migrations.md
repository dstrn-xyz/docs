# migrations

- [migrations](#migrations)
  - [introduction](#introduction)
  - [creating tables](#creating-tables)
  - [column types](#column-types)
  - [column modifiers](#column-modifiers)
  - [foreign keys](#foreign-keys)
  - [modifying tables](#modifying-tables)
  - [dropping tables](#dropping-tables)
  - [running migrations](#running-migrations)
    - [rolling back](#rolling-back)

<a name="introduction"></a>

## introduction

migrations are like version control for your database, allowing your team to define and share the application's database schema definition. if you have ever had to tell a teammate to manually add a column to their local database schema, you've faced the problem that database migrations solve.

each migration file is an es module that must export an asynchronous `up` function, and optionally a `down` function. the framework automatically injects a context object containing the `Schema` service, the `table` query builder, the raw `db` connection, and a `query` helper. these properties are also made available globally during the execution of the migration.

<a name="creating-tables"></a>

## creating tables

to create a new database table, use the `create` method on the `Schema` facade. the `create` method accepts two arguments: the name of the table and a closure which receives a `TableBuilder` object used to define the new table.

```javascript
export async function up({ Schema }) {
  await Schema.create('users', table => {
    table.increments('id');
    table.string('name');
    table.string('email').unique();
    table.timestamps();
  });
}

export async function down({ Schema }) {
  await Schema.dropIfExists('users');
}
```

<a name="column-types"></a>

## column types

the table builder contains a variety of column types that you may use when building your tables.

| type                                            | description                                     |
| ----------------------------------------------- | ----------------------------------------------- |
| `table.increments('id');`                       | auto incrementing integer primary key           |
| `table.bigIncrements('id');`                    | auto incrementing big integer primary key       |
| `table.integer('votes');`                       | integer column                                  |
| `table.bigInteger('views');`                    | big integer column                              |
| `table.float('amount');`                        | float column                                    |
| `table.decimal('balance', '10,2');`             | decimal column with precision                   |
| `table.boolean('confirmed');`                   | tinyint boolean column                          |
| `table.string('name', 100);`                    | varchar column with optional length             |
| `table.text('description');`                    | text column                                     |
| `table.json('options');`                        | json column                                     |
| `table.enum('status', ['active', 'inactive']);` | enum column                                     |
| `table.datetime('created_at');`                 | datetime column                                 |
| `table.date('birthday');`                       | date column                                     |
| `table.time('sunrise');`                        | time column                                     |
| `table.timestamps();`                           | adds created_at and updated_at datetime columns |

<a name="column-modifiers"></a>

## column modifiers

in addition to the column types listed above, there are several column modifiers you may use while adding a column to a database table.

| Type                                                 | Description                 |     |
| ---------------------------------------------------- | --------------------------- | --- |
| `table.string('email').nullable();`                  | allows null values          |
| `table.string('title').notNullable();`               | prevents null values        |
| `table.integer('votes').unsigned();`                 | makes integer unsigned      |
| `table.integer('status').defaultTo(1);`              | sets default value          |
| `table.string('email').unique();`                    | adds unique index           |
| `table.string('slug').index();`                      | adds basic index            |
| `table.integer('id').primary();`                     | explicitly adds primary key |
| `table.string('status').comment('the user status');` | adds column comment         |

<a name="foreign-keys"></a>

## foreign keys

dframework supports adding foreign key constraints to your tables. you define the local column using the `foreign` method, specify the referenced column and table using the `references` method, and optionally define cascade rules.

```javascript
export async function up({ Schema }) {
  await Schema.create('posts', table => {
    table.increments('id');
    table.integer('user_id').unsigned();
    
    table.foreign('user_id')
         .references('id', 'users')
         .onDelete('cascade');
  });
}
```

<a name="modifying-tables"></a>

## modifying tables

the `table` method on the `Schema` facade allows you to update existing tables.

```javascript
export async function up({ Schema }) {
  await Schema.table('users', table => {
    table.string('phone').nullable();
    table.index('phone');
  });
}
```

you can also rename an existing table using the `rename` method.

```javascript
await Schema.rename('old_table', 'new_table');
```

<a name="dropping-tables"></a>

## dropping tables

to drop an existing table, you may use the `drop` or `dropIfExists` methods.

```javascript
await Schema.drop('users');
await Schema.dropIfExists('users');
```

<a name="running-migrations"></a>

## running migrations

the framework runs migrations sequentially in alphabetical order based on the filenames in your migrations directory. a database table named `migrations` is automatically generated to track which migrations have already been executed, organizing them into batches.

this ensures that only pending migrations are executed, avoiding duplicate table creations.

<a name="rolling-back"></a>

### rolling back

if you need to revert the latest database changes, the framework allows you to roll back the most recent migration batch. it queries the `migrations` table to identify the last batch number and executes the `down` function for those specific files in reverse chronological order.
