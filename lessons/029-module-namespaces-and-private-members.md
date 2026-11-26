# 029 - Module Namespaces and Private Members

## Learning goals

After this lesson, you should be able to:

- Predict the default namespace for an `@use` rule.
- Choose an explicit namespace that keeps module references clear.
- Explain the trade-offs of `as *`.
- Distinguish file-level dependency scope from module-member visibility.
- Mark variables, mixins, and functions as private with a leading hyphen or underscore.
- Use `@forward` to expose a deliberate public API.
- Filter or prefix members forwarded from another module.

## 1. A namespace identifies where a member comes from

When a stylesheet loads another stylesheet with `@use`, its variables, functions, and mixins are accessed through a namespace by default.

~~~scss
// _colors.scss
$brand-color: #1769aa;

@mixin brand-border {
  border-color: $brand-color;
}
~~~

~~~scss
// app.scss
@use "colors";

.button {
  color: colors.$brand-color;
  @include colors.brand-border;
}
~~~

The references `colors.$brand` and `colors.contrast(...)` show which module provides each member. Module namespaces do not appear in the generated CSS; they organize Sass source code.

## 2. How Sass chooses the default namespace

By default, Sass uses the last component of the module URL, without its extension.

~~~scss
@use "design-system/colors";
~~~

The default namespace is `colors`. A URL such as `@use "helpers/color-functions"` gets the default namespace `color-functions`.

A default namespace may be unclear or may collide with another module's namespace. Give the module an alias in that case.

## 3. Choose an explicit namespace

~~~scss
@use "design-system/colors" as palette;
@use "legacy/colors" as legacy-palette;

.button {
  color: palette.$brand;
  border-color: legacy-palette.$border;
}
~~~

Both modules can expose short member names because their namespaces distinguish them. Aliases are local to the stylesheet containing these `@use` rules; another stylesheet chooses its own names.

Prefer aliases that communicate a module's role. A one-letter alias can work for a very common utility module, but an unexplained abbreviation makes references harder to scan.

## 4. What as * changes

The `as *` form removes the namespace:

~~~scss
@use "tokens" as *;

.card {
  padding: $space-medium;
  color: $text-color;
}
~~~

This can be convenient for a small module you own. It can also make a member's origin unclear and create naming conflicts as the project grows. A normal namespace keeps provenance explicit:

~~~scss
@use "tokens";

.card {
  padding: tokens.$space-medium;
  color: tokens.$text-color;
}
~~~

Use `as *` only when flattening that module's members is an intentional part of the local design. Avoid it for third-party or frequently changing dependencies.

## 5. Namespaces do not make members global across files

Every stylesheet has its own module scope. Loading a module in an entrypoint does not make that module's members available in every partial.

~~~scss
// components/_card.scss
@use "../tokens";

.card {
  padding: tokens.$space-medium;
}
~~~

This partial can be compiled through different entrypoints because it declares the module it needs. Do not rely on an entrypoint's `@use` to provide a hidden dependency to a child file.

## 6. Public and private module members

By default, a variable, mixin, or function that does not start with `-` or `_` is public. A member whose name starts with either character is private to the module that defines it.

~~~scss
// _buttons.scss
$-focus-color: #135cb5;

@function -focus-outline($color) {
  @return 3px solid $color;
}

@mixin -focus-style {
  outline: -focus-outline($-focus-color);
  outline-offset: 2px;
}

@mixin accessible-focus {
  @include -focus-style;
}
~~~

The module can use its private members internally, and consumers can use its public `accessible-focus` mixin:

~~~scss
@use "buttons";

.button:focus-visible {
  @include buttons.accessible-focus;
}
~~~

A consumer cannot access `buttons.$-focus-color` or include `buttons.-focus-style`. Keep implementation details private when consumers should use a higher-level public mixin or function instead.

## 7. Private members are not the same as partial filenames

A leading underscore in a **filename**, such as `_buttons.scss`, is Sass partial naming used by module resolution. A leading underscore or hyphen in a **member name**, such as `$-focus-color`, marks that variable as private.

These conventions can appear together, but they solve different problems:

- A partial filename is a file-loading convention.
- A private member name controls whether a module consumer can access that member.

Making a file a partial does not automatically make every variable in it private.

## 8. Use @forward to define a library entrypoint

A library can keep implementation files organized internally and expose a central entrypoint:

~~~text
sass/
  _index.scss
  _buttons.scss
  _forms.scss
  _tokens.scss
~~~

The entrypoint forwards selected modules:

~~~scss
// sass/_index.scss
@forward "buttons";
@forward "forms";
@forward "tokens";
~~~

A consumer loads the library entrypoint:

~~~scss
@use "sass" as ui;

.button {
  @include ui.accessible-focus;
  color: ui.$brand-color;
}
~~~

`@forward` makes the forwarded modules' public members available to consumers of the entrypoint. Forwarding does not make those members available inside `_index.scss` itself. If the entrypoint also needs a forwarded member internally, it should `@use` that module too.

## 9. Hide internal members from a forwarded API

Forwarding everything can expose more than a library intends. Use `hide` to exclude named members or `show` to allow only named members.

~~~scss
// sass/_index.scss
@forward "tokens" hide $internal-spacing-scale;
@forward "buttons" show accessible-focus, button-reset;
~~~

A downstream stylesheet using `sass` can access the forwarded public members but not the hidden variable or the unlisted button mixins.

This makes the entrypoint a deliberate API boundary. Private member names remain private automatically; `show` and `hide` help control which public members are re-exported.

## 10. Add a prefix while forwarding

A prefix can make forwarded member names more specific when the entrypoint combines multiple modules:

~~~scss
// sass/_index.scss
@forward "buttons" as button-*;
~~~

If `_buttons.scss` defines a public mixin named `reset`, a consumer loads and uses the forwarded name like this:

~~~scss
@use "sass" as ui;

.button {
  @include ui.button-reset;
}
~~~

A prefix changes the names consumers use through this forward. It does not rename the original member inside its defining module.

## 11. Forward and use the same module

An entrypoint may need to forward members to consumers and also use them itself:

~~~scss
// sass/_index.scss
@forward "buttons";
@use "buttons";

.button {
  @include buttons.accessible-focus;
}
~~~

Write `@forward` before `@use` when both load the same module. This lets downstream configuration be applied as the module is forwarded before the local `@use` loads it without configuration.

The `@use` provides the `buttons` namespace inside this file; the `@forward` exposes public members to users of the entrypoint.

## 12. API design guidelines

A useful module API has clear boundaries:

- Keep implementation helpers private when consumers should not call them directly.
- Expose stable, purposeful variables, functions, and mixins.
- Give library consumers one documented entrypoint when the library needs one.
- Use `@forward show` or `hide` to control that entrypoint.
- Add a forward prefix if it prevents ambiguity across modules.
- Avoid forwarding every internal file by default.

Renaming a public member may break consumers. Treat public names as part of the library's compatibility contract.

## 13. Common mistakes

### Using the wrong namespace

The namespace comes from the final component of the module URL unless an `as` alias is supplied. Check the exact `@use` line in the current file.

### Expecting an alias in one file to apply everywhere

Aliases are local to the stylesheet that declares them. Each file using a module can choose its own alias.

### Trying to access a private member

A leading `-` or `_` marks a Sass member as private. Expose a public mixin or function that performs the supported operation instead of reaching into the implementation.

### Assuming a partial file is private

The underscore in `_tokens.scss` does not hide members. Use member names with a leading `-` or `_` when they should be private.

### Expecting @forward to make a member available locally

`@forward` exposes members through the forwarding module to downstream users. Add an `@use` rule too if the forwarding stylesheet needs those members itself.

### Accidentally creating an overly broad public API

A plain `@forward` exposes all public members from the forwarded module. Use `show`, `hide`, and prefixes to publish only what consumers should rely on.

## 14. Summary

- A namespace identifies the module that provides a Sass member.
- Sass derives a default namespace from the module URL; `as` lets you choose another.
- `as *` removes the namespace but can create collisions and hide member origins.
- Members are scoped to the stylesheet that loads their module.
- A leading hyphen or underscore makes a Sass member private; a partial filename is a separate convention.
- `@forward` creates a downstream-facing API, and `show`, `hide`, and prefixes control that API.
- Use both `@forward` and `@use` when a file must expose a module and use its members locally.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="028-module-system-with-use.md">028 - The Module System with @use</a></td><td><a href="../qnas/029-module-namespaces-and-private-members.md">Q&amp;A</a></td><td><a href="../examples/029-module-namespaces-and-private-members.md">Example</a></td><td><a href="030-module-configuration-and-defaults.md">030 - Module Configuration and !default</a></td></tr>
  </tbody>
</table>
