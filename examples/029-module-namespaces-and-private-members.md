# 029 — Module Namespaces and Private Members: Examples

These examples show how Sass module names are referenced and how a library can shape its public API. File paths are included in comments; keep the same layout when compiling a multi-file example.

## 1. Use a module's default namespace

### styles/_colors.scss

~~~scss
$brand: #1769aa;

@mixin brand-border {
  border: 1px solid $brand;
}
~~~

### styles/app.scss

~~~scss
@use "colors";

.button {
  color: colors.$brand;
  @include colors.brand-border;
}
~~~

### CSS output

~~~css
.button {
  color: #1769aa;
  border: 1px solid #1769aa;
}
~~~

Sass uses `colors`, the last part of the module URL, as the default namespace.

## 2. Give modules with similar names explicit aliases

### File structure

~~~text
styles/
  design-system/
    _colors.scss
  legacy/
    _colors.scss
  app.scss
~~~

### styles/app.scss

~~~scss
@use "design-system/colors" as palette;
@use "legacy/colors" as legacy-palette;

.button {
  color: palette.$brand;
  border-color: legacy-palette.$border;
}
~~~

### Example module members

~~~scss
// design-system/_colors.scss
$brand: #1769aa;

// legacy/_colors.scss
$border: #8a94a6;
~~~

### CSS output

~~~css
.button {
  color: #1769aa;
  border-color: #8a94a6;
}
~~~

The aliases make the source of each member explicit even though both module URLs end in `colors`.

## 3. Flatten names only when it is intentional

### SCSS

~~~scss
@use "tokens" as *;

.card {
  padding: $space-medium;
  color: $text-color;
}
~~~

This removes the `tokens.` prefix. If another module loaded with `as *` also exposes `$text-color`, the source no longer tells you which module supplies it. For growing projects, prefer:

~~~scss
@use "tokens";

.card {
  padding: tokens.$space-medium;
  color: tokens.$text-color;
}
~~~

## 4. Use private implementation helpers

### styles/_focus.scss

~~~scss
$-ring-color: #135cb5;

@function -ring($width: 3px) {
  @return $width solid $-ring-color;
}

@mixin -focus-style {
  outline: -ring();
  outline-offset: 2px;
}

@mixin accessible-focus {
  @include -focus-style;
}
~~~

### styles/app.scss

~~~scss
@use "focus";

.button:focus-visible {
  @include focus.accessible-focus;
}
~~~

### CSS output

~~~css
.button:focus-visible {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}
~~~

The consumer can call `focus.accessible-focus`, but cannot access `focus.$-ring-color`, `focus.-ring()`, or `focus.-focus-style`. The public mixin is the supported interface.

## 5. A partial filename and a private member do different jobs

### styles/_tokens.scss

~~~scss
$public-brand: #1769aa;
$-internal-scale: 1.25;
~~~

The filename underscore marks `_tokens.scss` as a partial. The public variable is still visible to a stylesheet that loads the module; only `$-internal-scale` is private.

~~~scss
@use "tokens";

.example {
  color: tokens.$public-brand;
  // tokens.$-internal-scale is private and cannot be accessed here.
}
~~~

## 6. Forward a small library through one entrypoint

### File structure

~~~text
sass/
  _index.scss
  _buttons.scss
  _tokens.scss
~~~

### sass/_buttons.scss

~~~scss
@mixin reset {
  border: 0;
  font: inherit;
}

@mixin accessible-focus {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}
~~~

### sass/_tokens.scss

~~~scss
$brand-color: #1769aa;
$text-color: #263238;
$-internal-grid-unit: 0.25rem;
~~~

### sass/_index.scss

~~~scss
@forward "buttons";
@forward "tokens" show $brand-color, $text-color;
~~~

### styles/app.scss

~~~scss
@use "sass" as ui;

.button {
  @include ui.reset;
  @include ui.accessible-focus;
  color: ui.$text-color;
  background-color: ui.$brand-color;
}
~~~

### CSS output

~~~css
.button {
  border: 0;
  font: inherit;
  outline: 3px solid #135cb5;
  outline-offset: 2px;
  color: #263238;
  background-color: #1769aa;
}
~~~

The consumer loads the public `sass` entrypoint rather than depending directly on internal files. The forwarding file exposes the selected token variables and the public button mixins. The private grid unit is not forwarded.

## 7. Hide one public member from the entrypoint

### sass/_tokens.scss

~~~scss
$brand-color: #1769aa;
$text-color: #263238;
$legacy-brand: #0b4f7a;
~~~

### sass/_index.scss

~~~scss
@forward "tokens" hide $legacy-brand;
~~~

A consumer of `sass` can use `ui.$brand-color` and `ui.$text-color`, but `ui.$legacy-brand` is not part of the forwarded API.

Use `hide` when most members should be forwarded but a few should stay internal. Use `show` when only a small, explicit list should be public.

## 8. Add a prefix to forwarded names

### sass/_buttons.scss

~~~scss
@mixin reset {
  border: 0;
  font: inherit;
}

@mixin accessible-focus {
  outline: 3px solid #135cb5;
}
~~~

### sass/_index.scss

~~~scss
@forward "buttons" as button-*;
~~~

### styles/app.scss

~~~scss
@use "sass" as ui;

.button {
  @include ui.button-reset;
  @include ui.button-accessible-focus;
}
~~~

### CSS output

~~~css
.button {
  border: 0;
  font: inherit;
  outline: 3px solid #135cb5;
}
~~~

The prefix is part of the names exposed by `sass/_index.scss`. The original mixins inside `_buttons.scss` remain named `reset` and `accessible-focus`.

## 9. Forward a module and use it locally

### sass/_buttons.scss

~~~scss
@mixin accessible-focus {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}
~~~

### sass/_index.scss

~~~scss
@forward "buttons";
@use "buttons";

.button:focus-visible {
  @include buttons.accessible-focus;
}
~~~

### styles/app.scss

~~~scss
@use "sass";

.card:focus-visible {
  @include sass.accessible-focus;
}
~~~

The entrypoint uses `buttons` through the local namespace to define its own rule and also forwards its public mixin for downstream users. When both rules are compiled, both use the same mixin declarations.

~~~css
.button:focus-visible {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}

.card:focus-visible {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}
~~~

Place `@forward` before `@use` when both load the same module.

## 10. Keep a library's helper private while forwarding its API

### sass/_spacing.scss

~~~scss
$-base-unit: 0.25rem;

@function -steps($count) {
  @return $-base-unit * $count;
}

@function space($count) {
  @return -steps($count);
}
~~~

### sass/_index.scss

~~~scss
@forward "spacing";
~~~

### styles/app.scss

~~~scss
@use "sass" as ui;

.stack {
  gap: ui.space(4);
}
~~~

### CSS output

~~~css
.stack {
  gap: 1rem;
}
~~~

The entrypoint forwards the public `space()` function. Its helper function and base-unit variable remain private to `_spacing.scss`.

## 11. Compare direct module loading with a library entrypoint

### Direct dependency

~~~scss
@use "sass/buttons";
@use "sass/tokens";

.button {
  @include buttons.reset;
  color: tokens.$brand-color;
}
~~~

This couples the consumer to the library's internal module paths.

### Public entrypoint

~~~scss
@use "sass" as ui;

.button {
  @include ui.reset;
  color: ui.$brand-color;
}
~~~

With an entrypoint that forwards those names, consumers depend on the documented API. The library can reorganize its internal files while keeping the public entrypoint stable.

## 12. Check public and private references

Given this module:

~~~scss
// _colors.scss
$brand: #1769aa;
$-cache-key: "brand-v1";

@mixin brand-border {
  border-color: $brand;
}

@mixin -internal-border {
  border-color: $brand;
}
~~~

This consumer is valid:

~~~scss
@use "colors";

.button {
  @include colors.brand-border;
}
~~~

These references are invalid because they try to access private members:

~~~scss
@use "colors";

// Sass errors: these members are private.
.example {
  color: colors.$-cache-key;
  @include colors.-internal-border;
}
~~~

The invalid snippet is for comparison only; do not compile it as part of a working stylesheet.

## Further reading

- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
