# 028 — The Module System with @use: Examples

These examples use Dart Sass's module system. Multi-file examples show file names as comments; keep the same directory structure when compiling them.

## 1. Load variables and mixins through namespaces

### File structure

~~~text
styles/
  _tokens.scss
  _mixins.scss
  app.scss
~~~

### styles/_tokens.scss

~~~scss
$brand-color: #1769aa;
$text-color: #263238;
~~~

### styles/_mixins.scss

~~~scss
@mixin focus-ring {
  outline: 3px solid #9ac7f5;
  outline-offset: 2px;
}
~~~

### styles/app.scss

~~~scss
@use "tokens";
@use "mixins";

.button {
  color: tokens.$text-color;
  background-color: tokens.$brand-color;
  @include mixins.focus-ring;
}
~~~

### CSS output

~~~css
.button {
  color: #263238;
  background-color: #1769aa;
  outline: 3px solid #9ac7f5;
  outline-offset: 2px;
}
~~~

The members stay scoped to the module that defines them, and the namespace makes each dependency visible.

## 2. Give a module a short alias

### SCSS

~~~scss
@use "design-system/spacing" as space;

.panel {
  padding: space.$medium;
  gap: space.$small;
}
~~~

Assuming the module defines `$medium` and `$small`, the members are referenced through the chosen alias `space`. An alias is useful when a path is long or its default namespace would be unclear.

## 3. See why as * can cause collisions

### SCSS

~~~scss
@use "layout" as *;
@use "components/card" as *;

.page {
  padding: $space;
  color: $text;
}
~~~

If both modules expose `$space` or `$text`, the flattened names are ambiguous or conflict. Prefer explicit namespaces unless you own the modules and have verified that their public member names cannot collide.

## 4. A partial loads its own dependency

### File structure

~~~text
styles/
  _tokens.scss
  components/
    _card.scss
  app.scss
~~~

### styles/_tokens.scss

~~~scss
$surface: #ffffff;
$text-color: #263238;
$border-color: #cbd5e1;
~~~

### styles/components/_card.scss

~~~scss
@use "../tokens";

.card {
  color: tokens.$text-color;
  background-color: tokens.$surface;
  border: 1px solid tokens.$border-color;
  padding: 1rem;
}
~~~

### styles/app.scss

~~~scss
@use "components/card";
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #ffffff;
  border: 1px solid #cbd5e1;
  padding: 1rem;
}
~~~

The component declares the module it uses. It does not depend on the entrypoint loading `tokens` first.

## 5. Include module CSS only once

### styles/_reset.scss

~~~scss
* {
  box-sizing: border-box;
}
~~~

### styles/components/_button.scss

~~~scss
@use "../reset";

.button {
  padding: 0.5rem 0.75rem;
}
~~~

### styles/components/_card.scss

~~~scss
@use "../reset";

.card {
  padding: 1rem;
}
~~~

### styles/app.scss

~~~scss
@use "components/button";
@use "components/card";
~~~

### CSS output

~~~css
* {
  box-sizing: border-box;
}

.button {
  padding: 0.5rem 0.75rem;
}

.card {
  padding: 1rem;
}
~~~

Both components load the same reset module, but the reset's top-level CSS appears once in this compilation.

## 6. Configure a theme module from an entrypoint

### styles/_theme.scss

~~~scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;
$text-color: #263238 !default;
~~~

### styles/components/_button.scss

~~~scss
@use "../theme";

.button {
  color: #ffffff;
  background-color: theme.$brand-color;
}
~~~

### styles/app.scss

~~~scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa,
  $text-color: #241b35
);

@use "components/button";

.panel {
  color: theme.$text-color;
  background-color: theme.$surface-color;
}
~~~

### CSS output

~~~css
.button {
  color: #ffffff;
  background-color: #4b2e83;
}

.panel {
  color: #241b35;
  background-color: #f4f1fa;
}
~~~

The entrypoint configures `theme` before loading `components/button`. The button partial's own `@use "../theme"` sees the same configured module.

## 7. Compile separate themes in separate entrypoints

A single module has one configuration per Sass compilation. Separate entrypoints can produce separate themes:

### styles/light.scss

~~~scss
@use "theme" with (
  $brand-color: #1769aa,
  $surface-color: #ffffff,
  $text-color: #263238
);

@use "components/button";
~~~

### styles/dark.scss

~~~scss
@use "theme" with (
  $brand-color: #a8c7fa,
  $surface-color: #172033,
  $text-color: #f4f7fb
);

@use "components/button";
~~~

Compile `light.scss` and `dark.scss` as separate entrypoints. Each compilation loads the theme once with that entrypoint's configuration.

## 8. Use a built-in module

### SCSS

~~~scss
@use "sass:math";

$gutter: 24px;
$columns: 3;
$column-gap: math.div($gutter, $columns);

.grid {
  gap: $gutter;
}

.grid__item {
  padding-inline: $column-gap;
}
~~~

### CSS output

~~~css
.grid {
  gap: 24px;
}

.grid__item {
  padding-inline: 8px;
}
~~~

Built-in functions also use namespaces. For example, use `math.div()` instead of relying on the deprecated global `div()` function.

## 9. Resolve an index file through a module URL

### File structure

~~~text
styles/
  theme/
    _index.scss
    _colors.scss
    _spacing.scss
  app.scss
~~~

### styles/theme/_index.scss

~~~scss
@use "colors";
@use "spacing";

$surface: colors.$surface;
$gutter: spacing.$gutter;
~~~

### styles/theme/_colors.scss

~~~scss
$surface: #f4f7fb;
~~~

### styles/theme/_spacing.scss

~~~scss
$gutter: 1.5rem;
~~~

### styles/app.scss

~~~scss
@use "theme";

.panel {
  padding: theme.$gutter;
  background-color: theme.$surface;
}
~~~

### CSS output

~~~css
.panel {
  padding: 1.5rem;
  background-color: #f4f7fb;
}
~~~

Sass resolves `@use "theme"` through the directory's index file. The index file can load and expose members from its sibling modules for use in that file. The `@forward` rule is the usual tool for exposing a library's members through a public entrypoint.

## 10. Replace a simple Sass @import

### Before: legacy import

~~~scss
// _tokens.scss
$brand: #1769aa;

// app.scss
@import "tokens";

.button {
  background-color: $brand;
}
~~~

### After: module loading

~~~scss
// _tokens.scss
$brand: #1769aa;

// app.scss
@use "tokens";

.button {
  background-color: tokens.$brand;
}
~~~

The updated stylesheet uses an explicit namespace. A larger migration may require updating every dependent partial and revisiting imports that were nested inside style rules.

## 11. Load a module from a configured load path

Given this file structure:

~~~text
project/
  styles/
    app.scss
  shared-sass/
    _tokens.scss
~~~

The entrypoint can use a short module URL when `shared-sass` is configured as a load path:

~~~scss
// styles/app.scss
@use "tokens";

.page {
  color: tokens.$text;
}
~~~

The exact command-line or build-tool option for setting load paths depends on the Sass compiler integration. Keep load paths explicit in project configuration so another developer can resolve the same module URL.

## 12. Review a migration result

After running the Sass Migrator on an entrypoint, review these details:

- Does every partial load the modules whose members it uses?
- Are variables, functions, and mixins referenced through the intended namespaces?
- Are theme values configured before dependent modules load?
- Did the migration change where top-level CSS is emitted?
- Did any nested import need to become a mixin or a deliberate CSS-loading call?
- Does the compiled output preserve the intended selectors and source order?

The migration command is:

~~~sh
sass-migrator module --migrate-deps path/to/entrypoint.scss
~~~

The command updates Sass source. It does not decide whether the resulting module boundaries and public APIs fit the project's design.

## Further reading

- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: Breaking Change — @import and global built-in functions](https://sass-lang.com/documentation/breaking-changes/import/)
