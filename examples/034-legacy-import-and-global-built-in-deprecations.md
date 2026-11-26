# 034 - Legacy @import and Global Built-In Deprecations: Examples

These examples show how to distinguish Sass imports from CSS imports, replace global Sass dependencies with modules, and review a migration.

## 1. Replace global imports with namespaced modules

### Legacy files

~~~scss
// src/scss/_tokens.scss
$brand-color: #1769aa;

// src/scss/_button-tools.scss
@mixin base {
  border: 0;
  font: inherit;
}

// src/scss/app.scss
@import "tokens";
@import "button-tools";

.button {
  @include base;
  background-color: $brand-color;
}
~~~

The app stylesheet depends on both files, but their members enter one shared global scope.

### Module-based version

~~~scss
// src/scss/app.scss
@use "tokens";
@use "button-tools";

.button {
  @include button-tools.base;
  background-color: tokens.$brand-color;
}
~~~

Expected CSS:

~~~css
.button {
  border: 0;
  font: inherit;
  background-color: #1769aa;
}
~~~

The namespaces show which module owns each member.

## 2. Replace global built-in calls

### Before

~~~scss
$theme: (
  "brand": #1769aa,
  "text": #263238
);
$steps: (0.25rem, 0.5rem, 1rem);

$brand: map-get($theme, "brand");
$second-step: nth($steps, 2);
$unit-name: unit(12px);
$percentage-value: percentage(0.25);

.status {
  color: $brand;
  padding: $second-step;
  --unit-name: #{$unit-name};
  width: $percentage-value;
}
~~~

### After

~~~scss
@use "sass:list";
@use "sass:map";
@use "sass:math";

$theme: (
  "brand": #1769aa,
  "text": #263238
);
$steps: (0.25rem, 0.5rem, 1rem);

$brand: map.get($theme, "brand");
$second-step: list.nth($steps, 2);
$unit-name: math.unit(12px);
$percentage-value: math.percentage(0.25);

.status {
  color: $brand;
  padding: $second-step;
  --unit-name: #{$unit-name};
  width: $percentage-value;
}
~~~

Expected CSS:

~~~css
.status {
  color: #1769aa;
  padding: 0.5rem;
  --unit-name: px;
  width: 25%;
}
~~~

The built-in module import and namespace make the source of each function explicit.

## 3. Preserve browser-managed CSS imports

### Source

~~~scss
// app.scss
@import url("https://example.test/fonts.css");
@import "theme.css";

.page-title {
  font-family: sans-serif;
}
~~~

These are CSS imports. Sass leaves them in the generated CSS for the browser:

~~~css
@import url("https://example.test/fonts.css");
@import "theme.css";
.page-title {
  font-family: sans-serif;
}
~~~

Do not replace these with @use. Migrate Sass module imports, not browser-managed CSS URLs.

## 4. Convert a nested import to an opt-in mixin

### Before: nested Sass import

~~~scss
// _print.scss
.screen-only {
  display: block;
}

.print-only {
  display: none;
}

// app.scss
@media print {
  @import "print";
}
~~~

### After: preferred mixin pattern

~~~scss
// _print.scss
@mixin styles {
  .screen-only {
    display: none !important;
  }

  .print-only {
    display: block !important;
  }
}

// app.scss
@use "print";

@media print {
  @include print.styles;
}
~~~

Expected CSS:

~~~css
@media print {
  .screen-only {
    display: none !important;
  }
  .print-only {
    display: block !important;
  }
}
~~~

The module loads at the top level, and the mixin emits its CSS inside the media query.

## 5. Use meta.load-css() when the imported file cannot be changed

If an existing stylesheet cannot be edited to expose a mixin, load its CSS at the intended at-rule:

~~~scss
@use "sass:meta";

@media print {
  @include meta.load-css("print");
}
~~~

This is a direct option for loading CSS in a nested context. Prefer a mixin when you can control the module because the mixin makes the emitted styles and parameters explicit.

## 6. Configure a theme through @use

### Legacy global configuration

~~~scss
// src/scss/_theme.scss
$text-color: #263238 !default;
$surface-color: white !default;

// src/scss/components/_card.scss
.card {
  color: $text-color;
  background-color: $surface-color;
}

// src/scss/dark.scss
$text-color: white;
$surface-color: #20242a;

@import "theme";
@import "components/card";
~~~

The card module relies on variables that arrive through global imports.

### Module-based configuration

~~~scss
// src/scss/_theme.scss
$text-color: #263238 !default;
$surface-color: white !default;

// src/scss/components/_card.scss
@use "../theme";

.card {
  color: theme.$text-color;
  background-color: theme.$surface-color;
}

// src/scss/dark.scss
@use "theme" with (
  $text-color: white,
  $surface-color: #20242a
);

@use "components/card";
~~~

Expected CSS from dark.scss:

~~~css
.card {
  color: white;
  background-color: #20242a;
}
~~~

The component explicitly uses the theme module. It sees the configuration applied by the entrypoint when that module is first loaded.

## 7. Compare repeated Sass imports with module loading

### Legacy imports

~~~scss
// _base.scss
html {
  box-sizing: border-box;
}

// _header.scss
@import "base";

.site-header {
  padding: 1rem;
}

// _footer.scss
@import "base";

.site-footer {
  padding: 1rem;
}

// app.scss
@import "header";
@import "footer";
~~~

A legacy Sass import evaluates the stylesheet at each import site, so the html rule can be emitted repeatedly through the dependency chain.

### Module-based source

~~~scss
// _base.scss
html {
  box-sizing: border-box;
}

// _header.scss
@use "base";

.site-header {
  padding: 1rem;
}

// _footer.scss
@use "base";

.site-footer {
  padding: 1rem;
}

// app.scss
@use "header";
@use "footer";
~~~

With the Sass module system, the shared base module is loaded once in the app.scss compilation. Its html rule appears once in that output.

## 8. Preview a migration before changing files

Install the migrator in the project:

~~~bash
npm install --save-dev sass-migrator
~~~

Preview changes to the application and its dependencies:

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

The dry run prints proposed changes without writing them. Review changes such as:

- New @use rules.
- Added member namespaces.
- Changes to configured variables.
- Changes to nested imports.
- Changes to public members or legacy prefixes.

When the proposed changes look right, apply the migration:

~~~bash
npx sass-migrator module --migrate-deps src/scss/app.scss
~~~

Review the changed Sass source and compile it with the project's normal build command.

## 9. Pass load paths to the migrator

If the Sass build resolves modules from a shared directory, the migrator needs the same load path to read those files.

~~~bash
npx sass-migrator --load-path=src/scss module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Dependencies loaded through load paths are treated as dependencies by the migrator. Review which source files are maintained by the project and which belong to third-party packages.

## 10. Migrate only global built-ins as an intermediate step

If imports cannot be migrated in the same change, update built-in function calls first:

~~~bash
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

This migration can turn map-get($theme, "brand") into map.get($theme, "brand") while leaving Sass @import rules in the source. The @import deprecation still needs its own follow-up.

## 11. Preserve a legacy import API with an import-only file

A library can offer a modern module API and a temporary compatibility API for existing @import consumers.

~~~scss
// _reset.scss
@mixin list {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }
}

// _reset.import.scss
@forward "reset" as reset-*;
~~~

A new consumer uses the module namespace:

~~~scss
// new-app.scss
@use "reset";

@include reset.list;
~~~

A legacy consumer can keep its old prefixed mixin name:

~~~scss
// old-app.scss
@import "reset";

@include reset-list;
~~~

The .import.scss file is used by @import, not by @use. This compatibility layer can help a library transition its API, but the legacy consumer still uses a deprecated Sass import.

## 12. Identify warning sources

A source file may contain both deprecated patterns:

~~~scss
@import "theme";
$brand: map-get($palette, "brand");
~~~

They produce two different kinds of warnings:

- The Sass import warning identifies the legacy dependency-loading rule.
- The global built-in warning identifies a built-in function called without its sass: namespace.

A targeted migration can address either pattern first. Check the warning text and Sass's deprecation identifier before choosing a fix.

## 13. Temporarily silence dependency warnings

If a third-party Sass dependency cannot yet be updated, --quiet-deps can reduce warning output from dependencies in a CLI build:

~~~bash
npx sass --quiet-deps src/scss/app.scss dist/app.css
~~~

This does not change the dependency source. Do not use it to conceal warnings from application code you own.

For a temporary targeted silence of both deprecation types, pass both identifiers to Dart Sass:

~~~bash
npx sass \
  --silence-deprecation=import \
  --silence-deprecation=global-builtin \
  src/scss/app.scss dist/app.css
~~~

Keep silencing scoped to a real compatibility need and track the remaining migration work.

## 14. Practice tasks

1. Replace a Sass @import of a token partial with @use and namespace the variables.
2. Replace map-get(), nth(), and unit() with functions from sass:map, sass:list, and sass:math.
3. Keep an @import url(...) rule unchanged while migrating a Sass import beside it.
4. Convert a nested Sass import to a mixin included inside @media.
5. Preview a module migration with --dry-run and inspect the changed namespaces.
6. Run a built-in-only migration and identify which @import warnings remain.
7. Add an import-only compatibility file for one old library mixin and document its temporary status.

## Quick reference

~~~scss
@use "tokens";
@use "sass:map";
@use "sass:list";
@use "sass:math";

.card {
  color: tokens.$brand-color;
}

$brand: map.get($theme, "brand");
$second: list.nth($steps, 2);
$unit-name: math.unit(12px);
~~~

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
npx sass-migrator module --migrate-deps src/scss/app.scss
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

## Related material

- [Lesson: Legacy @import and global built-in deprecations](../lessons/034-legacy-import-and-global-built-in-deprecations.md)
- [Q&A: Legacy @import and global built-in deprecations](../qnas/034-legacy-import-and-global-built-in-deprecations.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="033-emitted-css-and-entrypoints.md">033 - Generated CSS and Entry Points: Examples</a></td><td><a href="../lessons/034-legacy-import-and-global-built-in-deprecations.md">Lesson</a></td><td><a href="../qnas/034-legacy-import-and-global-built-in-deprecations.md">Q&amp;A</a></td><td><a href="035-migrating-to-sass-modules.md">035 - Migrating to Sass Modules: Examples</a></td></tr>
  </tbody>
</table>
