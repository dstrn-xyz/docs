# バリデーション

- [バリデーション](#バリデーション)
  - [はじめに](#はじめに)
  - [基本的な使用法](#基本的な使用法)
  - [エラーの操作](#エラーの操作)
  - [カスタムメッセージ](#カスタムメッセージ)
  - [利用可能なルール](#利用可能なルール)
  - [ファイルバリデーション](#ファイルバリデーション)

<a name="はじめに"></a>

## はじめに

dframeworkはパイプ区切りのルールセットに対して受信データを評価するグローバルに利用可能な`validate()`ヘルパーを提供します。バリデーターは標準のデータオブジェクトと受信マルチパートファイルアップロードの両方を処理します。

<a name="基本的な使用法"></a>

## 基本的な使用法

第1引数に検証したいデータを渡し第2引数にルールを含むオブジェクトを渡します。ファイルを検証する場合は`req.body`だけでなく`req`オブジェクト全体を渡す必要があります。

いずれかのルールに違反した場合`fails()`メソッドは`true`を返します。

```javascript
export default class AuthController {
  async register(req) {
    const validation = validate(req.body, {
      username: 'required|string|min:3|max:20',
      email:    'required|email',
      password: 'required|string|min:8'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }

    // proceed with registration
  }
}
```

<a name="エラーの操作"></a>

## エラーの操作

バリデーション結果オブジェクトはエラーメッセージを操作するための複数のメソッドを提供します。

- `fails()`: バリデーションが失敗した場合にtrueを返します
- `passes()`: バリデーションが成功した場合にtrueを返します
- `errors()`: フィールド名をエラーメッセージの配列にマッピングしたオブジェクトを返します
- `first(field)`: 特定のフィールドの最初のエラーメッセージを返します
- `all()`: フィールド名をエラーメッセージの配列にマッピングしたオブジェクトを返します（errors()と同じです）

jsonではなくhtmlレスポンスを返す場合は通常`.withErrors()`および`.withInput()`ヘルパーを使用します。

```javascript
export default class SettingsController {
  async update(req) {
    const validation = validate(req.body, {
      display_name: 'nullable|string|max:50'
    });

    if (validation.fails()) {
      return back().withErrors(validation.errors()).withInput(req.body);
    }
  }
}
```

<a name="カスタムメッセージ"></a>

## カスタムメッセージ

カスタムエラーメッセージを定義するには第3引数としてオブジェクトを渡します。キーには`field.rule`のドット表記を使用する必要があります。

```javascript
const messages = {
  'email.required': 'we really need your email address',
  'email.email': 'that does not look like a valid email',
  'password.min': 'your password must be at least 8 characters long'
};

const validation = validate(req.body, rules, messages);
```

<a name="利用可能なルール"></a>

## 利用可能なルール

組み込みルールは最も一般的なバリデーションシナリオを網羅しています。

| ルール | 説明 |
| --- | --- |
| `required` | フィールドが存在し空でない必要があります |
| `nullable` | フィールドは任意です（空の場合ルールの位置に関係なくそのフィールドのバリデーションルールはスキップされます） |
| `string` | フィールドは文字列である必要があります |
| `number` | フィールドは有効な数値（整数または浮動小数点数）である必要があります |
| `integer` | フィールドは有効な整数である必要があります |
| `boolean` | フィールドはブール値（`true`、`false`、`1`、`0`、`yes`、`no`）である必要があります |
| `array` | フィールドは配列である必要があります |
| `email` | フィールドは有効なメール形式である必要があります |
| `min:value` | 文字列または配列の場合: 最小長。数値の場合: 最小値 |
| `max:value` | 文字列または配列の場合: 最大長。数値の場合: 最大値 |
| `in:foo,bar` | フィールドは指定された値のいずれかである必要があります |
| `not_in:foo,bar` | フィールドは指定された値のいずれでもない必要があります |

<a name="ファイルバリデーション"></a>

## ファイルバリデーション

> [!IMPORTANT]
> アップロードされたファイルを検証する際はバリデーターが`req.files`を検査できるように`req.body`ではなく完全な`req`オブジェクトを`validate(req, rules)`に渡してください。

ファイルサイズはバイト単位または省略記法サフィックス（`kb`、`mb`、`gb`）で指定できます。

```javascript
export default class UserProfileController {
  async updateProfileImage(req) {
    const validation = validate(req, {
      profileImage: 'file_required|file_size:8mb|file_max_count:1|file_mimes:jpg,png,jpeg'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }
  }
}
```

利用可能なファイルバリデーションルールは以下のとおりです。

| ルール | 説明 |
| --- | --- |
| `file` | フィールドはファイルの配列である必要があります（必須） |
| `file_required` | フィールドは少なくとも1つのアップロードファイルを含む必要があります |
| `file_size:size` | 個々のファイルが指定されたサイズ（例: `8mb`、`500kb`）を超えることはできません |
| `file_min_size:size` | 個々のファイルが指定されたサイズ未満になることはできません |
| `file_mimes:types` | すべてのファイルがいずれかの拡張子またはmimeタイプ（例: `jpg,png,pdf`）に一致する必要があります |
| `file_max_count:n` | ファイル数が`n`を超えることはできません |
| `file_min_count:n` | ファイル数が少なくとも`n`である必要があります |
