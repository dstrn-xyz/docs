# d-otp

form associated one time password and verification code input handled by a single native input for  SMS autofill, paste sanitization, keyboard editing, and segmented slot groups.

```html
<d-otp length="6" separator="3" name="code"></d-otp>
<d-otp length="4" mask name="pin"></d-otp>
<d-otp length="6" pattern="alphanumeric" uppercase separator="2"></d-otp>
```

<a name="attributes-and-properties-d-otp"></a>
## attributes and properties

| attribute   | property    | type      | default    | form | description                                                                        |
| :---------- | :---------- | :-------- | :--------- | :--- | :--------------------------------------------------------------------------------- |
| `length`    | `length`    | `number`  | `6`        | no   | total number of character slots (1 to 16)                                          |
| `value`     | `value`     | `string`  | `""`       | yes  | sanitized OTP string reflected in form submission                                  |
| `name`      | `name`      | `string`  | `""`       | no   | form field name                                                                    |
| `pattern`   | `pattern`   | `string`  | `"digits"` | no   | allowed character preset (`"digits"`, `"alphanumeric"`, `"alpha"`) or regex string |
| `separator` | `separator` | `string`  | `""`       | no   | group split interval (e.g. `"3"`) or comma separated split indices (e.g. `"2,4"`)  |
| `mask`      | `mask`      | `boolean` | `false`    | no   | masks entered characters as dots for PIN entry                                     |
| `disabled`  | `disabled`  | `boolean` | `false`    | no   | disables editing and excludes the field from form submission                       |
| `invalid`   | `invalid`   | `boolean` | `false`    | no   | applies error border styling to all slots                                          |
| `autofocus` | `autofocus` | `boolean` | `false`    | no   | focuses the OTP input automatically on mount                                       |
| `uppercase` | `uppercase` | `boolean` | `false`    | no   | transforms entered letters to uppercase before validation                          |

<a name="programmatic-api-d-otp"></a>
## programmatic api

```javascript
const otp = select('d-otp');

otp.focus();
otp.error('invalid verification code');
otp.clearError();
otp.clear();
```

<a name="events-d-otp"></a>
## events

| event      | detail           | bubbles | description                                         |
| :--------- | :--------------- | :------ | :-------------------------------------------------- |
| `input`    | string (`value`) | yes     | dispatched whenever the sanitized OTP value changes |
| `change`   | string (`value`) | yes     | dispatched when the OTP value updates               |
| `complete` | string (`value`) | yes     | dispatched once whenever all slots are filled       |

<a name="behavior-d-otp"></a>
## behavior

- uses a single underlying `<input autocomplete="one-time-code">` so mobile autofill, clipboard paste (automatically stripping separators or non matching characters), and backspace work normally.
