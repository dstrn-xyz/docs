# views

- [views](#views)
  - [introduction](#introduction)
  - [creating views](#creating-views)
  - [displaying data](#displaying-data)
  - [control structures](#control-structures)
  - [layouts and includes](#layouts-and-includes)
  - [javascript execution](#javascript-execution)
  - [localization](#localization)
  - [pagination](#pagination)
    - [custom pagination html](#custom-pagination-html)
  - [automatic form handling](#automatic-form-handling)
  - [anti peeping protection](#anti-peeping-protection)
  - [view compilation](#view-compilation)

<a name="introduction"></a>

## introduction

dframework ships with a custom view engine. it uses a string to function evaluation model that compiles your templates into raw javascript functions. this means you can write standard javascript directly inside your templates while utilizing convenient directives for common tasks.

<a name="creating-views"></a>

## creating views

views are stored in the `views` directory and must use the `.d` file extension. they are referenced using dot notation. a file at `views/app/home/index.d` is referenced as `app.home.index`.

```html
<!-- views/greetings/hello.d -->
<html>
    <body>
        <h1>hello, {{ name }}!</h1>
    </body>
</html>
```

you return views from your controllers using the `render()` helper.

```javascript
export default class GreetingController {
  async hello() {
    return render('greetings.hello', { name: 'world' });
  }
}
```

for security reasons, you cannot pass dangerous javascript global objects (`process`, `require`, `global`, etc.) into your view data. the engine strictly validates the provided context and will throw an error if tampering is detected.

<a name="displaying-data"></a>

## displaying data

you display data passed to your views by wrapping the variable in double curly braces. the view engine automatically escapes the output to prevent xss attacks.

```html
hello, {{ user.name }}.
the current timestamp is {{ Date.now() }}.
```

if you need to render unescaped html, use the `{!! !!}` syntax.

```html
{!! user.bio_html !!}
```

you can convert objects to json strings using the `@json` directive.

```html
<script>
    const userState = @json(user);
</script>
```

<a name="control-structures"></a>

## control structures

the view engine provides shortcuts for common javascript control structures.

**if statements**

```html
@if (user.isAdmin)
    <button>admin panel</button>
@elseif (user.isModerator)
    <button>mod panel</button>
@else
    <span>standard user</span>
@endif
```

**loops**

```html
@foreach (users as user)
    <li>{{ user.name }}</li>
@endforeach

@for (let i = 0; i < 10; i++)
    <span>{{ i }}</span>
@endfor
```

the `@forelse` directive is a convenient way to loop over arrays with a fallback if the array is empty.

```html
@forelse (tracks as track)
    <li>{{ track.title }}</li>
@empty
    <li>no tracks found.</li>
@endforelse
```

<a name="layouts-and-includes"></a>

## layouts and includes

you can define a master layout and extend it from child views. use `@yield` to mark where content should be injected, and `@section` in the child view to provide that content.

```html
<!-- views/layouts/app.d -->
<html>
    <head>
        <title>app</title>
    </head>
    <body>
        <nav>...</nav>
        <main>
            @yield('content')
        </main>
    </body>
</html>
```

```html
<!-- views/home.d -->
@extends('layouts.app')

@section('content')
    <h1>welcome home</h1>
@endsection
```

you can include partials using the `@include` directive.

```html
@include('components.header')
```

<a name="javascript-execution"></a>

## javascript execution

since the view engine compiles directly to javascript functions, you can execute arbitrary server side code directly inside your templates. use the `@js()` directive for a single line of javascript, or the `@js ... @endjs` block for multiple lines.

```html
@js(const maxLimit = 50;)

@js
    let sortedUsers = users.sort((a, b) => b.score - a.score);
    sortedUsers = sortedUsers.slice(0, maxLimit);
@endjs

@foreach (sortedUsers as user)
    <li>{{ user.name }}</li>
@endforeach
```

<a name="localization"></a>

## localization

you can translate strings using the `@t()` directive. the framework automatically resolves the current locale from the request context and fetches the corresponding translation string.

```html
<h1>@t('home.welcome_message')</h1>
<p>@t('home.unread_count', { count: 5 })</p>
```

<a name="pagination"></a>

## pagination

the view engine includes a built in `@pagination` directive that automatically generates html pagination links for a paginated result object returned by the model `.paginate()` method.

if you leave the block empty, the engine automatically generates standard pagination controls with icons and page numbers.

```html
<!-- renders standard pagination controls -->
@pagination(users)
@endpagination
```

by default, if the request comes from the spa router (`dSPAHttpRequest`), the links will automatically trigger spa navigation instead of a full page reload. if you need the pagination links to update a specific dom element via the spa router (e.g. replacing just a table body), you can pass a css selector as the third argument.

```html
<!-- updates the #user-list container instead of a full page transition -->
@pagination(users, true, '#user-list')
@endpagination
```

### custom pagination html

if you provide content inside the `@pagination` block, the engine skips the default html generation and renders your custom html instead.

inside the block, the engine automatically exposes four variables that you can use to build your custom controls:

- `prev`: the query string for the previous page (or `null`)
- `next`: the query string for the next page (or `null`)
- `current`: the current page number
- `last`: the total number of pages

```html
@pagination(users)
    <div class="custom-pagination">
        <p>page {{ current }} of {{ last }}</p>
        
        @if (prev)
            <a href="{{ prev }}" class="btn">previous</a>
        @endif
        
        @if (next)
            <a href="{{ next }}" class="btn">next</a>
        @endif
    </div>
@endpagination
```

the framework evaluates the block only if there is more than one page (`last > 1`). if there is only one page, the entire `@pagination` block is skipped and nothing is rendered.

<a name="automatic-form-handling"></a>

## automatic form handling

when you create an html `<form>` with a mutating method (e.g. `POST`, `PUT`), the view engine automatically injects a hidden `_csrf` input field and the current csrf token value. you do not need to manually output csrf tokens in your forms.

additionally, the engine injects a `<meta name="csrf-token">` tag into the document's `<head>`, allowing your ajax requests to pick up the token.

<a name="anti-peeping-protection"></a>

## anti peeping protection

dframework includes a built in anti peeping script that disables context menus, blocks common developer tools keyboard shortcuts, and triggers infinite debugger loops if the console is forced open.

this behavior can be toggled globally via `app.antiPeeping` in your configuration, or overridden per response using the `antiPeeping(false)` chainable helper in your controller.

<a name="view-compilation"></a>

## view compilation

in the `local` environment, the framework uses a custom jit compiler and provides detailed error traces mapped directly to your original `.d` files.

in the `production` environment, views are ahead of time compiled into highly optimized `.dc` (.d compiled) files. the framework verifies the cryptographic integrity of these `.dc` files on startup to ensure your templates have not been tampered with.