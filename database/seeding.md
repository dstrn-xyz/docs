# seeding

- [seeding](#seeding)
  - [introduction](#introduction)
  - [model factories](#model-factories)
  - [writing seeders](#writing-seeders)
  - [running seeders](#running-seeders)
  - [automatic safety checks](#automatic-safety-checks)

<a name="introduction"></a>

## introduction

seeding allows you to populate your newly migrated database tables with initial or dummy data, completing the setup process for new environments instantly. dframework provides model factories for declarative test and seed data generation alongside transaction safe seeders.

<a name="model-factories"></a>

## model factories

factories live in `database/factories/` and extend `Factory` from `'dframework'`. generate one with the cli:

```bash
dstrn make:factory UserFactory --model=User
```

define `definition(fake)` returning default attributes using the built in `fake` helper (`fake.uuid()`, `fake.name()`, `fake.email()`, `fake.sentence()`, `fake.number()`, `fake.pick()`, etc.), plus optional custom state methods:

```javascript
import { Factory } from 'dframework';
import User from '../../models/User.js';

export default class UserFactory extends Factory {
  static model = User;

  definition(fake) {
    return {
      name: fake.name(),
      email: fake.email(),
      role: 'member'
    };
  }

  admin() {
    return this.state({ role: 'admin' });
  }
}
```

invoke factories directly from any `Model` class via `Model.factory(count?)`:

```javascript
// create and persist a single user
const user = await User.factory().create();

// create 10 users with alternating roles
const users = await User.factory(10)
  .sequence({ role: 'editor' }, { role: 'viewer' })
  .create();

// build unsaved instances in memory
const draft = await User.factory().admin().make({ name: 'preview' });

// compose belongsTo (.for) and hasMany (.has) relationships
const posts = await Post.factory(3).for(User.factory()).create();
const author = await User.factory().has(Post.factory(5)).create();

// lifecycle hooks
const active = await User.factory()
  .afterMaking((user) => { user.status = 'active'; })
  .afterCreating(async (user) => { await user.load('posts'); })
  .create();
```

<a name="writing-seeders"></a>

## writing seeders

seeder files live in the `database/seeders/` directory and are responsible for populating tables with data. similar to migrations, a seeder file must export an asynchronous `run` function. the framework injects a context object containing table, db, query, and call (to run other seeders idempotently in the same transaction), giving you direct access to insert operations.

these context properties are also available globally during execution.

```javascript
export async function run({ table, call }) {
  await call('RoleSeeder');
  await table('users').insert([
    {
      name: 'admin',
      email: 'admin@example.com',
      password: 'password' // automatically hashed
    }
  ]);
}
```

<a name="running-seeders"></a>

## running seeders

to execute all available seeders in sequence, execute the seed command:

```bash
dstrn seed
```

to run a specific individual seeder, pass the seeder name as an argument:

```bash
dstrn seed UserSeeder
dstrn seed 001_UserSeeder
```

when executing in production (`APP_ENV=production`), the seed command prompts for confirmation to prevent accidental data changes. use the `--force` flag for automated provisioning pipelines:

```bash
dstrn seed --force
```

<a name="automatic-safety-checks"></a>

## automatic safety checks

the seeder runner is designed for safety in production environments.

> [!NOTE]
> before executing an insert statement within a seeder, the framework performs a database count on the target table. if records exist, the runner rolls back the transaction for that seeder and skips the entire seeder file, allowing seeder suites to run repeatedly without primary key collisions or duplicate rows.