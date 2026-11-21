# d-form

ajax form container with automatic csrf token injection, client side validation, inline field error rendering, and response routing.

```html
<d-form action="/register" method="POST" rules='{"email": "required|email"}' class="flex-column g-1">
  <input type="email" name="email" placeholder="email address">
  <input type="password" name="password" placeholder="password">
  <button type="submit" class="btn">register</button>
</d-form>
```

<a name="attributes-and-properties-d-form"></a>

## attributes and properties

| attribute      | property      | type      | default  | form | description                                                        |
| :------------- | :------------ | :-------- | :------- | :--- | :----------------------------------------------------------------- |
| `action`       | `action`      | `string`  | `""`     | no   | submission destination url                                         |
| `method`       | `method`      | `string`  | `"POST"` | no   | http request method (`GET`, `POST`, `PUT`, `DELETE`)               |
| `rules`        | `rules`       | `string`  | `""`     | no   | json string of client side validation rules                        |
| `target`       | `target`      | `string`  | `""`     | no   | css selector of element to swap when server returns html           |
| `navigate`     | `navigate`    | `boolean` | `false`  | no   | updates browser history and uses spa router when html is returned  |
| `force-reload` | `forceReload` | `string`  | `""`     | no   | reload condition (`""`, `"true"`, `"all"`, `"success"`, `"error"`) |
| `callback`     | `callback`    | `string`  | `""`     | no   | name of global window function executed after successful response  |

<a name="programmatic-api-d-form"></a>

## programmatic api

```javascript
const form = select('d-form');

// submit form programmatically
form.submit();

// retrieve form data
const formData = form.serialize();

// display validation errors manually
form.showErrors({ email: ['invalid address'] });

// clear active errors
form.clearErrors();

// disable or enable form controls
form.setDisabled(true);
```

<a name="events-d-form"></a>

## events

| event        | detail               | bubbles | description                                              |
| :----------- | :------------------- | :------ | :------------------------------------------------------- |
| `d-submit`   | none                 | yes     | cancelable, dispatched before submission starts          |
| `d-success`  | `object` or `string` | yes     | dispatched when request succeeds                         |
| `d-error`    | `object`             | yes     | dispatched when request fails or validation errors occur |
| `d-response` | `object` or `string` | yes     | dispatched on every completed response                   |

<a name="behavior-d-form"></a>

## behavior

- wraps or injects an internal form element and intercepts the submit event.
- injects `X-Requested-With: dSPAHttpRequest` and `X-CSRF-TOKEN` headers automatically.
- disables all inputs, buttons, and form controls during in flight submission to prevent duplicate requests.
- evaluates client side rules if defined (prevents submission and injects inline field errors if validation fails).
- for file uploads, sends `multipart/form-data` (for standard fields, sends `application/x-www-form-urlencoded`).

<a name="response-lifecycle"></a>

## response lifecycle

| controller return                  | status / type                             | default behavior (no force-reload)                                                                                                   | behavior with force-reload                                                                    |
| :--------------------------------- | :---------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------- |
| `return json({ data })`            | `200` (`application/json`)                | parses payload, dispatches `d-success` and `d-response`, executes `callback` if defined, leaves form on page without reload          | reloads page if set to `force-reload`, `all`, or `success` (ignored on `error`)               |
| `return json({ redirect })`        | `200` (`application/json`)                | navigates via `dSPA.navigate(redirect)` without reload (executes full page redirect if `force: true` or `X-Redirect-Force` is set)   | navigates via spa router (or hard redirect if `force: true`)                                  |
| `return json().withErrors(errors)` | `422` (`application/json`)                | marks inputs with `.error`, injects `.error-message` below fields (or `.error-banner` at top), dispatches `d-error` and `d-response` | reloads page if set to `force-reload`, `all`, or `error` (retains inline errors on `success`) |
| `validate()` failure               | `422` (`application/json`)                | renders field validation errors inline, dispatches `d-error` and `d-response`, keeps form state intact                               | reloads page if set to `force-reload`, `all`, or `error` (retains inline errors on `success`) |
| `return abort(status, message)`    | `4xx` or `5xx` (`application/json`)       | displays `.error-banner` at top of form, dispatches `d-error` and `d-response`, reenables form inputs                                | reloads page if set to `force-reload`, `all`, or `error` (displays banner on `success`)       |
| `return render(view, data)`        | `200` (`text/html`)                       | swaps returned markup into target selector (default `body`), runs scripts, dispatches `d-success` (updates url if `navigate` is set) | reloads page if set to `force-reload`, `all`, or `success` (proceeds with swap on `error`)    |
| unexpected server error            | `500` (`text/html` or `application/json`) | renders full 500 error page via `document.write` for html (displays error banner for json)                                           | displays 500 error page for html (reloads page for json if set to `all` or `error`)           |
| unexpected 404 error               | `404` (`text/html` or `application/json`) | swaps target with 404 error template for html (displays error banner for json)                                                       | reloads page if set to `all` or `error` (preserves view on `success`)                         |
