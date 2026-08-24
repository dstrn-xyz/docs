# seeding

- [seeding](#seeding)
  - [introduction](#introduction)
  - [writing seeders](#writing-seeders)
  - [automatic safety checks](#automatic-safety-checks)

<a name="introduction"></a>

## introduction

seeding allows you to populate your newly migrated database tables with initial or dummy data, completing the setup process for new environments instantly. dframework provides a simple mechanism for defining and executing these seeding operations.

<a name="writing-seeders"></a>

## writing seeders

seeder files live in the seeders directory and are responsible for populating tables with data. similar to migrations, a seeder file must export an asynchronous `run` function. the framework injects a context object containing the fluent `table` query builder, giving you direct access to insert operations.

these context properties are also available globally during execution.

```javascript
export async function run({ table }) {
  await table('users').insert([
    {
      name: 'admin',
      email: 'admin@example.com',
      password: 'password' // automatically hashed
    }
  ]);
}
```

<a name="automatic-safety-checks"></a>

## automatic safety checks

the seeder runner is designed for safety in production environments.

> [!NOTE]
> before executing an insert statement within a seeder, the framework performs a database count on the target table. if records exist, the insert is safely skipped, allowing seeder suites to run repeatedly without primary key collisions or duplicate rows.