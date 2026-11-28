# シーディング

- [シーディング](#シーディング)
  - [はじめに](#はじめに)
  - [モデルファクトリ](#モデルファクトリ)
  - [シーダーの記述](#シーダーの記述)
  - [シーダーの実行](#シーダーの実行)
  - [自動安全性チェック](#自動安全性チェック)

<a name="はじめに"></a>

## はじめに

シーディングを使用するとマイグレーション済みのデータベーステーブルに初期データやダミーデータを投入でき新しい環境のセットアップを即座に完了できます。dframeworkはトランザクションで保護されたシーダーに加えて宣言的なテスト・シードデータ生成のためのモデルファクトリを提供します。

<a name="モデルファクトリ"></a>

## モデルファクトリ

ファクトリは`database/factories/`に配置し`'dframework'`からインポートした`Factory`を継承します。CLIで生成できます:

```bash
dstrn make:factory UserFactory --model=User
```

組み込みの`fake`ヘルパー（`fake.uuid()`、`fake.name()`、`fake.email()`、`fake.sentence()`、`fake.number()`、`fake.pick()`など）を使ってデフォルト属性を返す`definition(fake)`と任意のカスタム状態メソッドを定義します:

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

任意の`Model`クラスから`Model.factory(count?)`でファクトリを呼び出せます:

```javascript
// 単一ユーザーを作成して永続化
const user = await User.factory().create();

// 交互のロールで10人のユーザーを作成
const users = await User.factory(10)
  .sequence({ role: 'editor' }, { role: 'viewer' })
  .create();

// 永続化せずメモリ上にインスタンスを生成
const draft = await User.factory().admin().make({ name: 'preview' });

// belongsTo（.for）およびhasMany（.has）リレーションの構築
const posts = await Post.factory(3).for(User.factory()).create();
const author = await User.factory().has(Post.factory(5)).create();

// ライフサイクルフック
const active = await User.factory()
  .afterMaking((user) => { user.status = 'active'; })
  .afterCreating(async (user) => { await user.load('posts'); })
  .create();
```

<a name="シーダーの記述"></a>

## シーダーの記述

シーダーファイルは`database/seeders/`ディレクトリに配置しテーブルへのデータ投入を担当します。マイグレーションと同様にシーダーファイルは非同期の`run`関数をエクスポートする必要があります。フレームワークはtable、db、query、および同一トランザクション内で他のシーダーを冪等に実行するcallを含むコンテキストオブジェクトを注入し挿入操作への直接アクセスを提供します。

これらのコンテキストプロパティは実行中にグローバルでも利用可能です。

```javascript
export async function run({ table, call }) {
  await call('RoleSeeder');
  await table('users').insert([
    {
      name: 'admin',
      email: 'admin@example.com',
      password: 'password'
    }
  ]);
}
```

<a name="シーダーの実行"></a>

## シーダーの実行

利用可能なすべてのシーダーを順番に実行するにはseedコマンドを実行します:

```bash
dstrn seed
```

特定のシーダーを個別に実行するにはシーダー名を引数として渡します:

```bash
dstrn seed UserSeeder
dstrn seed 001_UserSeeder
```

本番環境（`APP_ENV=production`）で実行する場合seedコマンドは誤ったデータ変更を防ぐために確認を求めます。自動プロビジョニングパイプラインでは`--force`フラグを使用します:

```bash
dstrn seed --force
```

<a name="自動安全性チェック"></a>

## 自動安全性チェック

シーダーランナーは本番環境での安全性を考慮して設計されています。

> [!NOTE]
> シーダー内でinsert文を実行する前にフレームワークは対象テーブルのレコード数をカウントします。レコードが存在する場合ランナーはそのシーダーのトランザクションをロールバックしシーダーファイル全体をスキップします。これにより主キーの衝突や重複行を発生させることなくシーダースイートを繰り返し安全に実行できます。
