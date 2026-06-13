# validation

- [validation](#validation)
  - [introduction](#introduction)
  - [basic usage](#basic-usage)
  - [working with errors](#working-with-errors)
  - [custom messages](#custom-messages)
  - [available rules](#available-rules)
  - [file validation](#file-validation)

<a name="introduction"></a>

## introduction

dframework provides a globally available `validate()` helper that evaluates incoming data against a set of pipe separated rules. the validator handles both standard data objects and incoming multipart file uploads.

<a name="basic-usage"></a>

## basic usage

you pass the data you want to validate as the first argument and an object containing the rules as the second argument. to validate files, you must pass the entire `req` object instead of just `req.body`.

the `fails()` method returns `true` if any rule was violated.

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

<a name="working-with-errors"></a>

## working with errors

the validation result object provides several methods for interacting with the error messages:

- `fails()`: returns true if validation failed
- `passes()`: returns true if validation succeeded
- `errors()`: returns an object mapping field names to an array of error messages
- `first(field)`: returns the first error message for a specific field
- `all()`: returns a flat array of all error messages across all fields

when returning an html response rather than json, you typically use the `.withErrors()` and `.withInput()` helpers.

```javascript
export default class SettingsController {
  async update(req) {
    const validation = validate(req.body, {
      display_name: 'nullable|string|max:50'
    });

    if (validation.fails()) {
      return back().withErrors(validation.errors()).withInput(req.body);
    }
    
    // ...
  }
}
```

<a name="custom-messages"></a>

## custom messages

you can provide an object as the third argument to define custom error messages. the keys must use the `field.rule` dot notation.

```javascript
const messages = {
  'email.required': 'we really need your email address',
  'email.email': 'that does not look like a valid email',
  'password.min': 'your password must be at least 8 characters long'
};

const validation = validate(req.body, rules, messages);
```

<a name="available-rules"></a>

## available rules

the built in rules cover the most common validation scenarios:

| rule             | description                                                        |
| ---------------- | ------------------------------------------------------------------ |
| `required`       | the field must be present and not empty                            |
| `nullable`       | the field is optional; if empty, subsequent rules are skipped      |
| `string`         | the field must be a string                                         |
| `number`         | the field must be a valid number (integer or float)                |
| `integer`        | the field must be a valid integer                                  |
| `boolean`        | the field must be boolean (`true`, `false`, `1`, `0`, `yes`, `no`) |
| `array`          | the field must be an array                                         |
| `email`          | the field must be a valid email format                             |
| `min:value`      | for strings/arrays: minimum length. for numbers: minimum value     |
| `max:value`      | for strings/arrays: maximum length. for numbers: maximum value     |
| `in:foo,bar`     | the field must be one of the specified values                      |
| `not_in:foo,bar` | the field must not be one of the specified values                  |

<a name="file-validation"></a>

## file validation

to validate uploaded files, you must pass the entire `req` object to the validator instead of `req.body`. the framework reads the uploaded files from `req.files` automatically.

file sizes can be specified in bytes, or with shorthand suffixes (`kb`, `mb`, `gb`).

```javascript
export default class UserProfileController {
  async updateProfileImage(req) {
    const validation = validate(req, {
      profileImage: 'file_required|file_size:8mb|file_max_count:1|file_mimes:jpg,png,jpeg'
    });

    if (validation.fails()) {
      return status(422).json({ success: false, errors: validation.errors() });
    }
    
    // ...
  }
}
```

the available file validation rules are:

| rule                 | description                                                                   |
| -------------------- | ----------------------------------------------------------------------------- |
| `file`               | the field must be an array of files (optional)                                |
| `file_required`      | the field must contain at least one uploaded file                             |
| `file_size:size`     | no individual file can exceed the specified size (e.g. `8mb`, `500kb`)        |
| `file_min_size:size` | no individual file can be smaller than the specified size                     |
| `file_mimes:types`   | all files must match one of the extensions or mime types (e.g. `jpg,png,pdf`) |
| `file_max_count:n`   | the number of files cannot exceed `n`                                         |
| `file_min_count:n`   | the number of files must be at least `n`                                      |
