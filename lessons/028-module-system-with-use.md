# 028 - The Module System with @use

## Learning goals

After this lesson, you should be able to:

- Load another Sass stylesheet with `@use`.
- Access its variables, functions, and mixins through a module namespace.
- Explain how module CSS is included in compiled output.
- Configure a module through public `!default` variables.
- Use built-in Sass modules with explicit namespaces.
- Find Sass modules using partials, index files, and load paths.
- Recognize why `@use` is not a direct, global replacement for every `@import`.

## 1. Why Sass has a module system

Legacy Sass `@import` shared members in one global scope. As a project grew, a variable or mixin could be hard to trace, names could collide, and importing the same stylesheet repeatedly could repeat its CSS.

The module system uses `@use` to make dependencies explicit. A stylesheet loaded with `@use` is a **module**. Each stylesheet must load the modules whose members it uses, and those members are accessed through a namespace.

The next lessons examine namespaces, visibility, forwarding, and migration in more depth. This lesson establishes the core `@use` workflow.

## 2. Load a stylesheet and use its members

Suppose a project has these files:

~~~text
styles/
  _tokens.scss
  _mixins.scss
  app.scss
~~~

The token module defines a variable, and the mixin module defines reusable declarations:

~~~scss
// styles/_tokens.scss
$brand-color: #1769aa;

// styles/_mixins.scss
@mixin focus-ring {
  outline: 3px solid #9ac7f5;
  outline-offset: 2px;
}
~~~

The entrypoint loads both modules:

~~~scss
// styles/app.scss
@use "tokens";
@use "mixins";

.button {
  color: white;
  background-color: tokens.$brand-color;
  @include mixins.focus-ring;
}
~~~

By default, Sass derives each namespace from the final part of the module URL. In this example, the namespaces are `tokens` and `mixins`.

The compiled CSS is:

~~~css
.button {
  color: white;
  background-color: #1769aa;
  outline: 3px solid #9ac7f5;
  outline-offset: 2px;
}
~~~

The namespace makes the origin of each member visible in the source. `tokens.$brand-color` is a variable, while `mixins.focus-ring` is included as a mixin.

## 3. Choose a namespace when needed

A module can be loaded under an explicit alias:

~~~scss
@use "design-system/colors" as colors;
@use "design-system/spacing" as space;

.panel {
  color: colors.$text;
  padding: space.$medium;
}
~~~

Aliases are useful when a default namespace is unclear, too long, or conflicts with another module. Choose names that stay understandable across the file.

Sass also permits `@use "module" as *`, which exposes members without a namespace. This can reintroduce name collisions, so reserve it for cases where you control the module and have a clear reason to flatten its names.

## 4. A module's members belong to the stylesheet that loads it

Loading a module in one file does not make its members global to every other file. If a component partial uses a token, it should load the token module itself:

~~~scss
// styles/components/_card.scss
@use "../tokens";

.card {
  color: tokens.$brand-color;
}
~~~

This direct dependency makes `_card.scss` understandable when it is compiled through a different entrypoint. Do not rely on another file having loaded `tokens` first.

Names beginning with `-` or `_` are private to the module that defines them. Private-member design and public APIs are covered in the module namespaces lesson.

## 5. Module CSS is included once

A module can contain CSS as well as variables and mixins:

~~~scss
// styles/_reset.scss
* {
  box-sizing: border-box;
}
~~~

If multiple modules in one compilation load `reset`, Sass includes that module's CSS only once in the compiled output. Its members still need to be loaded in every stylesheet that uses them.

This makes module loading different from copying a file's contents at every reference. Keep CSS in a module only when loading that module should intentionally add those styles. Token, function, and mixin modules often emit no CSS until their members are used.

## 6. Place @use rules before stylesheet rules

An `@use` rule belongs at the top level of a stylesheet and must appear before style rules and other rules, except `@forward`. Sass variables may be declared before `@use` when they are used to configure a module.

~~~scss
@use "tokens";
@use "mixins";

.page {
  @include mixins.focus-ring;
}
~~~

This is invalid because `@use` appears inside a style rule:

~~~scss
.page {
  @use "tokens";
}
~~~

Load dependencies at the beginning of each file so readers can see what that file relies on.

## 7. Configure a module with !default variables

A module can expose settings with defaults:

~~~scss
// styles/_theme.scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;

.card {
  color: $brand-color;
  background-color: $surface-color;
}
~~~

An entrypoint can configure those public variables when it loads the module:

~~~scss
// styles/app.scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa
);
~~~

The configured values are used by the module's CSS. Configuration applies to variables explicitly marked `!default`. It must be provided when the module is first loaded in that compilation; a later `@use` cannot reconfigure an already-loaded module.

This pattern lets an application customize a library without editing the library's source. Keep configuration near the entrypoint so the application's choices are easy to find.

## 8. Use Sass's built-in modules

Modern Sass built-in functions are organized into modules. Load the module you need and use its namespace:

~~~scss
@use "sass:math";
@use "sass:color";

$column-width: math.div(12rem, 3);
$muted-blue: color.adjust(#1769aa, $lightness: 20%);

.column {
  width: $column-width;
  background-color: $muted-blue;
}
~~~

The math and color operations are explicit in the source. This is clearer than relying on legacy global function names. Dart Sass deprecated Sass `@import` and global built-in function calls starting in version 1.80.0; use the built-in module namespace for current code.

## 9. How Sass finds a module

Module URLs are usually written without the leading underscore used for partial filenames and without the `.scss` extension:

~~~scss
@use "tokens";
@use "components/card";
~~~

Sass resolves these URLs using its module resolution rules. Common matches include regular Sass files, partial files such as `_tokens.scss`, and a folder's `_index.scss`. A configured load path can make shared module URLs resolvable without long relative paths.

A relative URL is resolved from the stylesheet that contains the `@use`. Keep file organization and URL conventions consistent so dependencies remain easy to trace. Package URLs and custom importers have additional rules; consult the Sass documentation when using them.

## 10. Migrating from @import

`@use` does not preserve all of `@import`'s behavior:

- `@use` creates a module namespace instead of adding every member to a global scope.
- Each stylesheet must explicitly load the modules whose members it uses.
- `@use` must be at the top level, so an import nested inside a selector needs a different design.
- A module's CSS is included once per compilation, rather than each time its file is imported.

Dart Sass deprecated Sass `@import` in version 1.80.0. For a project-wide migration, the official Sass Migrator can update imports and references:

~~~sh
sass-migrator module --migrate-deps path/to/entrypoint.scss
~~~

Review the resulting changes. The migrator can update dependencies, but the project still needs review for nested imports, intended CSS placement, module configuration, and public library APIs.

## 11. Common mistakes

### Using a member without its namespace

If a stylesheet writes `$brand-color` but loaded the module as `tokens`, use `tokens.$brand-color` or deliberately choose another namespace.

### Assuming @use is global

A module loaded in `app.scss` is not automatically available inside `components/_card.scss`. Each stylesheet declares its own dependencies.

### Loading @use after CSS rules

Move `@use` to the top level before style rules and other non-`@forward` rules. A module cannot be loaded from inside a selector.

### Configuring a module too late

Provide `with (...)` the first time the module is loaded. If an earlier dependency already loaded it, Sass cannot apply a new configuration later in the same compilation.

### Expecting every loaded module to be CSS-free

A module may emit CSS when loaded. Check whether it intentionally contains top-level style rules before adding it to an entrypoint.

### Treating the Sass Migrator as a full code review

Run the migrator on the intended entrypoint, then inspect the result and compile the application. Automated rewriting cannot decide whether the old global behavior was part of the design.

## 12. Practical workflow

When adding a dependency:

1. Identify the file that directly uses the variable, mixin, or function.
2. Add an `@use` for the module to that file.
3. Use the module namespace at each reference.
4. Configure public `!default` variables in the entrypoint before other dependencies load the module.
5. Check whether the module emits CSS.
6. Compile and review the generated CSS when loading or configuring a module changes output.

## 13. Summary

- `@use` loads a Sass stylesheet as a module.
- Module members are accessed through a namespace by default.
- Each stylesheet explicitly loads its own dependencies.
- Module CSS is included only once per compilation.
- Public `!default` variables can be configured when the module is first loaded.
- Built-in modules such as `sass:math` provide namespaced functions.
- Sass `@import` is deprecated in Dart Sass; migration needs a review of the resulting module structure.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="027-at-root-and-at-rules.md">027 - @at-root and Nested At-Rules</a></td><td><a href="../qnas/028-module-system-with-use.md">Q&amp;A</a></td><td><a href="../examples/028-module-system-with-use.md">Example</a></td><td><a href="029-module-namespaces-and-private-members.md">029 - Module Namespaces and Private Members</a></td></tr>
  </tbody>
</table>
