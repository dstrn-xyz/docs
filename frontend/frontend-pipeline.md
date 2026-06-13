# frontend pipeline

- [introduction](#introduction)
- [the build pipeline](#the-build-pipeline)
- [view compilation](#view-compilation)
- [css optimization and obfuscation](#css-optimization-and-obfuscation)
- [route compilation](#route-compilation)

<a name="introduction"></a>

## introduction

unlike traditional frameworks that rely on external tools like webpack or vite, dframework includes its own highly specialized, zero configuration build pipeline. the pipeline is designed to eliminate unused css, heavily obfuscate class names, and compile your views and routes into extremely fast, cryptographically verified representations.

<a name="the-build-pipeline"></a>

## the build pipeline

the pipeline runs automatically when your application boots in the `production` environment. it calculates a fingerprint of your `views` and `routes` directories. if the fingerprint has not changed since the last build (tracked in `storage/framework/pipeline.json`), the pipeline instantly loads the cached artifacts, reducing startup time to milliseconds.

if changes are detected, the pipeline executes its three main phases: view compilation, css optimization, and route compilation.

<a name="view-compilation"></a>

## view compilation

the first phase compiles all `.d` view templates into raw javascript functions (`.dc` files). these compiled files are saved to `storage/framework/views`.

during this phase, a `manifest.json` file is generated containing a cryptographic hmac signature for every compiled view. at runtime, the framework verifies these signatures to ensure your compiled views have not been tampered with.

<a name="css-optimization-and-obfuscation"></a>

## css optimization and obfuscation

once the views are compiled, the pipeline scans the `.dc` files and your public javascript files to extract every css class name used in your project.

the css optimizer then processes the framework's utility css with the following steps:

1. **tree shaking**: any utility class not found in your views or javascript is completely removed.
2. **obfuscation**: every kept utility class is renamed to a short, random character sequence (e.g. `mt-4` becomes `ab`).
3. **rewriting**: the optimizer directly edits your compiled `.dc` views and public javascript files, replacing the original class names with their obfuscated counterparts.
4. **integrity updates**: the `manifest.json` is automatically updated with the new cryptographic signatures of the modified `.dc` files.
5. **minification**: the base css, obfuscated utilities, icons, components, and your custom css are bundled and minified into a single `public/css/dstrn.css` file.

the optimizer successfully identifies and rewrites classes within standard `class=""` attributes, dynamic bindings like `:class`, `x-bind:class`, and `v-bind:class`, as well as javascript `classList` operations.

a `css-map.json` file is written to the `storage/framework` directory for debugging purposes, mapping the original class names to their obfuscated versions.

### opting out of css optimization

if you prefer to skip css minification, tree shaking, and obfuscation entirely, you can opt out by updating your `config/app.js` configuration.

```javascript
export default {
  // ...
  css: {
    optimize: Env.value('OPTIMIZE_CSS', true),
  },
};
```

when optimization is disabled, the pipeline will still combine your core and user css files into a single `dstrn.css` payload, but it will not obfuscate or rewrite class names.

<a name="route-compilation"></a>

## route compilation

finally, the `RouteCompiler` scans your route definitions and their corresponding controller handlers. it analyzes the source code to determine exactly which middleware, dependencies, and payload types each route requires.

this data is compiled into a highly optimized route cache (`storage/framework/routes`), completely skipping runtime reflection or dependency resolution on incoming http requests.
