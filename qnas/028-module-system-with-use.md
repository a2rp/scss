# 028 - The Module System with @use: Q&A

## Fundamentals

### 1. What is a Sass module?

A module is a Sass stylesheet loaded with `@use` or `@forward`. It can provide variables, functions, mixins, and CSS.

### 2. What does @use do?

`@use` loads another stylesheet as a module. It makes that module's public members available to the current stylesheet through a namespace and includes its CSS in the compilation if the module emits CSS.

### 3. Why does Sass use modules instead of one global namespace?

Namespaces make member origins explicit and help avoid collisions. Each stylesheet also declares the dependencies it directly uses, making those dependencies easier to locate.

### 4. How do I use a variable from a module?

Load the module and write its namespace before the variable:

~~~scss
@use "tokens";

.card {
  color: tokens.$text-color;
}
~~~

### 5. How do I include a mixin from a module?

Use the namespace before the mixin name:

~~~scss
@use "mixins";

.button {
  @include mixins.focus-ring;
}
~~~

### 6. How do I call a function from a module?

Use the namespace before the function call. For example, after `@use "sass:math"`, call `math.div(12px, 3)`.

### 7. How is the default namespace chosen?

Sass uses the last part of the module URL, without its extension. For example, `@use "design-system/colors"` has the default namespace `colors`.

### 8. Can I choose a different namespace?

Yes. Write `@use "design-system/colors" as palette;` and use members such as `palette.$brand`.

### 9. What does as * do?

It loads a module's members without a namespace. This can create collisions with other members in the stylesheet, so a namespace is usually clearer. Sass recommends using `as *` only for modules you control.

### 10. Does @use make a module's members global to every Sass file?

No. A member is available only in the stylesheet that loads the module. Another partial that uses that member needs its own `@use`.

## CSS and loading

### 11. Is a module allowed to contain CSS?

Yes. A module can contain top-level CSS as well as variables, mixins, and functions. That CSS is included when the module is loaded.

### 12. How many times is a module's CSS emitted if the same module is loaded repeatedly?

Once per Sass compilation, even if multiple stylesheets load that module. Members still need to be loaded in each stylesheet that uses them.

### 13. Does loading a module emit CSS from every mixin in it?

No. A mixin's declarations are emitted when that mixin is included. Top-level style rules in the module are emitted when the module is loaded.

### 14. Where can @use appear?

It must be at the top level and before style rules and other rules, except `@forward`. Variables may appear before it when they are used to configure a module.

### 15. Can I put @use inside a style rule?

No. Load the module at the top level of the stylesheet. If old code uses a nested Sass `@import`, migrating it may require wrapping generated styles in a mixin or using another deliberate CSS-loading pattern.

### 16. How does @use find a partial?

Sass resolves the module URL using its module resolution rules. A URL like `@use "tokens"` can resolve to a partial such as `_tokens.scss`; the underscore and extension are usually omitted in the URL.

### 17. Can I load an index file from a directory?

Yes. Sass can resolve a directory module through its index file, commonly `_index.scss`. For example, `@use "theme"` can load `theme/_index.scss` when that is the matching module.

### 18. What is a load path?

A load path is a directory Sass searches for modules in addition to paths resolved relative to the current stylesheet. It can shorten shared module URLs. Configure it in the Sass CLI or build tool you use.

## Configuration

### 19. What does !default do for a module setting?

It gives a variable a default value that a consumer can override through module configuration.

~~~scss
// _theme.scss
$brand-color: #1769aa !default;
~~~

### 20. How do I configure a module?

Pass values in a `with (...)` clause on its `@use`:

~~~scss
@use "theme" with (
  $brand-color: #4b2e83
);
~~~

The configured variable must be public and declared with `!default`.

### 21. Can I configure a module after another dependency has loaded it?

No. Configuration must be supplied the first time that module is loaded in the compilation. If a dependency loaded the module earlier, a later configuration cannot change it.

### 22. Do every module's repeated @use calls get the same configuration?

Yes. A module is loaded once in a compilation and all users share that loaded module and its configuration. This supports configuring a theme in an entrypoint and having dependent components use that same theme.

### 23. Can one entrypoint build two different theme configurations at once?

A module can only have one configuration in a single Sass compilation. Build separate themes through separate entrypoints or compilations if they need different configurations.

### 24. Can I configure any variable from a module?

No. Configuration is for public variables that the module marks with `!default`. Private variables are not part of the consumer configuration API.

## Built-in modules and migration

### 25. How do I use Sass built-in functions in modern code?

Load the built-in module and use its namespace:

~~~scss
@use "sass:math";

$third: math.div(1, 3);
~~~

Other built-in modules include `sass:color`, `sass:list`, `sass:map`, `sass:meta`, `sass:selector`, and `sass:string`.

### 26. Is Sass @import the same as CSS @import?

No. Historically, Sass `@import` loaded Sass members into a shared global scope. A plain CSS `@import` is a browser CSS at-rule and is not a Sass module load.

### 27. Is Sass @import deprecated?

Yes. Dart Sass deprecated Sass `@import` and global built-in function calls in version 1.80.0. Sass recommends the module system for Sass dependencies.

### 28. What does the Sass Migrator do?

The module migrator can rewrite Sass imports and member references to use the module system. It is a starting point; review the changes for nested imports, CSS placement, configuration, and library APIs.

### 29. What command runs the module migrator?

After installing the `sass-migrator` package, run the module migration for an entrypoint:

~~~sh
sass-migrator module --migrate-deps path/to/entrypoint.scss
~~~

The `--migrate-deps` option includes dependencies of the entrypoint in the migration.

### 30. Is migration always a one-to-one replacement of @import with @use?

No. The module system has explicit namespaces and top-level loading rules, and it emits module CSS once. Old code that relies on global variables, repeated CSS output, or nested imports may need a design change.

### 31. What is the difference between @use and @forward?

`@use` makes a module's members available to the current stylesheet. `@forward` exposes a module's public members through the current stylesheet for downstream users; it does not make those forwarded members available inside the forwarding stylesheet by itself. The forwarding rules belong to the library API workflow.

## Troubleshooting

### 32. Sass says a module member is undefined. What should I check?

Check that the current file has an `@use` for the module, that you used the correct namespace, and that the member is public. Also check the spelling and module URL.

### 33. Why did another partial not see a module I loaded in app.scss?

Modules are not global. Add an `@use` to the partial that directly needs the member.

### 34. Why does Sass say a @use rule is in the wrong position?

Move it to the top level before style rules and other rules, except `@forward`. A variable declaration may come before `@use` when it configures the module.

### 35. Why did my module's CSS appear in the compiled file?

The loaded module contains top-level CSS or loads another module that emits CSS. Inspect the module graph and keep top-level CSS only where loading that module should add styles.

### 36. Why did a configured variable not change?

Confirm the variable is public and marked `!default`, that the configuration name matches exactly, and that no earlier dependency loaded the module without configuration.

### 37. Why did a file that used to work after @import lose access to a variable?

With `@use`, each file must explicitly load its dependency and use the right namespace. The old global variable may also have belonged to an import order that should now be replaced by module configuration.

### 38. What should I do if a migrated nested import no longer compiles?

`@use` cannot be nested. Refactor the nested module's CSS into a mixin and include it at the intended location, or use an appropriate CSS-loading approach such as `sass:meta`'s `load-css()` mixin. Review the output because parent-selector behavior and CSS placement may differ.

## Quick reference

| Need | Pattern |
| --- | --- |
| Load a module | `@use "tokens";` |
| Access a variable | `tokens.$brand-color` |
| Include a mixin | `@include mixins.focus-ring;` |
| Choose an alias | `@use "tokens" as t;` |
| Configure public defaults | `@use "theme" with ($brand-color: purple);` |
| Use a built-in module | `@use "sass:math";` |
| Expose members downstream | `@forward "tokens";` |
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="027-at-root-and-at-rules.md">027 - @at-root and Nested At-Rules: Q&amp;A</a></td><td><a href="../lessons/028-module-system-with-use.md">Lesson</a></td><td><a href="../examples/028-module-system-with-use.md">Example</a></td><td><a href="029-module-namespaces-and-private-members.md">029 - Module Namespaces and Private Members: Q&amp;A</a></td></tr>
  </tbody>
</table>
