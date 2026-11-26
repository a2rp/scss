# 034 - Legacy @import and Global Built-In Deprecations

## Learning goals

After this lesson, you should be able to:

- Tell deprecated Sass @import rules apart from plain CSS @import rules.
- Explain why Sass is moving code from the global scope to @use and @forward.
- Replace global built-in calls with namespaced functions from sass: modules.
- Migrate a nested Sass import to a mixin or meta.load-css().
- Use the Sass Migrator to preview and apply module migrations.
- Review behavior changes caused by replacing global imports with modules.
- Handle third-party deprecation warnings without hiding migration work in your own code.

## 1. What is deprecated?

Dart Sass deprecated Sass's @import rule and global calls to Sass built-in functions in version 1.80.0. The deprecation provides a warning period so projects and libraries can migrate to the Sass module system.

A deprecation warning does not mean that every existing stylesheet stops compiling immediately. It signals that code should be updated while the feature is still supported, before a future major release can remove it.

The module system uses:

- @use to load a module and access its members in the current stylesheet.
- @forward to expose members through a library entrypoint.
- Built-in module namespaces such as sass:map and sass:math for Sass's built-in functions.

These changes address different patterns, but they are often handled during the same modernization effort.

## 2. Sass @import and CSS @import are different

Historically, Sass @import could load Sass files at compile time, sharing variables, mixins, and functions globally and combining multiple stylesheets' CSS.

~~~scss
// Legacy Sass import
@import "tokens";
@import "components/button";
~~~

The Sass @import rule is deprecated. New Sass code should use @use and @forward.

A browser-oriented CSS import is different:

~~~scss
// Plain CSS imports remain browser-managed:
@import url("https://example.test/fonts.css");
@import "theme.css";
~~~

These CSS imports are preserved for the browser to handle. Do not mechanically change every line beginning with @import. Identify whether it is a Sass module import or an intentional CSS import first.

## 3. Why global Sass imports cause problems

A Sass @import makes variables, mixins, and functions available in a shared global scope. This can make ownership unclear:

~~~scss
// _colors.scss
$brand-color: #1769aa;

// app.scss
@import "colors";

.button {
  color: $brand-color;
}
~~~

From app.scss alone, it is not obvious where $brand-color came from. If multiple imports define the same name, later assignments or definitions can affect other files unexpectedly.

Repeated imports can also re-evaluate stylesheets and emit CSS repeatedly. This makes output and dependency behavior harder to understand.

With @use, members are accessed through an explicit namespace and each module is loaded once per compilation:

~~~scss
// app.scss
@use "colors";

.button {
  color: colors.$brand-color;
}
~~~

The reference shows which module owns the member.

## 4. Replace Sass imports with @use

### Before: global Sass imports

~~~scss
// _tokens.scss
$brand-color: #1769aa;

// _button-tools.scss
@mixin base {
  border: 0;
  font: inherit;
}

// app.scss
@import "tokens";
@import "button-tools";

.button {
  @include base;
  background-color: $brand-color;
}
~~~

### After: namespaced modules

~~~scss
// app.scss
@use "tokens";
@use "button-tools";

.button {
  @include button-tools.base;
  background-color: tokens.$brand-color;
}
~~~

The module URLs identify dependencies. The namespace comes from the URL by default, or it can be chosen with as:

~~~scss
@use "button-tools" as buttons;

.button {
  @include buttons.base;
}
~~~

If two module URLs would produce the same default namespace, assign explicit namespaces:

~~~scss
@use "components/button/mixins" as button-mixins;
@use "components/form/mixins" as form-mixins;
~~~

## 5. Replace global built-in function calls with module calls

Sass built-in functions historically had global names. Their module-system equivalents are namespaced by the module they belong to.

### Before

~~~scss
$theme: ("brand": #1769aa);
$steps: (0.25rem, 0.5rem, 1rem);

$brand: map-get($theme, "brand");
$second-step: nth($steps, 2);
$unit-name: unit(12px);
$percentage-value: percentage(0.25);
~~~

### After

~~~scss
@use "sass:list";
@use "sass:map";
@use "sass:math";

$theme: ("brand": #1769aa);
$steps: (0.25rem, 0.5rem, 1rem);

$brand: map.get($theme, "brand");
$second-step: list.nth($steps, 2);
$unit-name: math.unit(12px);
$percentage-value: math.percentage(0.25);
~~~

The namespace makes it clear whether a function operates on a map, list, or number. Check the built-in module documentation when replacing less common functions; a similarly named function may have a different signature or semantics.

## 6. Global built-in warnings are separate from Sass @import warnings

There are separate deprecations for:

- Sass @import rules.
- Global calls to Sass built-in functions that are available in sass: modules.

For example, a stylesheet can have one warning for @import and a different warning for map-get(). Updating only one pattern does not automatically fix the other.

~~~scss
// Both patterns need migration:
@import "tokens";
$value: map-get($theme, "brand");
~~~

~~~scss
// Module-based versions:
@use "sass:map";
@use "tokens";

$value: map.get($theme, "brand");
~~~

A project may also have other Sass deprecations, such as old color-function or division patterns. Read the warning identifier and relevant Sass documentation instead of treating all warnings as the same issue.

## 7. Keep plain CSS imports intact

These are examples of CSS imports rather than Sass module dependencies:

~~~scss
@import url("print.css");
@import url("https://example.test/fonts.css");
@import "theme.css";
~~~

Sass preserves CSS imports for the browser to handle. Do not convert them to @use: @use is for Sass modules and is not a replacement for browser-managed CSS URL loading.

A URL with an explicit .css extension is CSS-oriented. Inspect the source and compiler output to determine whether an import is a CSS import or a legacy Sass import.

## 8. Replace nested Sass imports deliberately

Legacy Sass allowed imports inside rules:

~~~scss
@media print {
  @import "print";
}
~~~

@use rules must be at the top level of a stylesheet, so this cannot be converted by simply moving @use into the media block.

### Preferred pattern: expose a mixin

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

The mixin makes the emitted CSS explicit and places it at the include site.

### Alternative: load CSS with sass:meta

If you cannot edit the imported stylesheet to wrap its CSS in a mixin, sass:meta provides load-css():

~~~scss
@use "sass:meta";

@media print {
  @include meta.load-css("print");
}
~~~

Use this when a direct module-loading pattern is needed inside a nested context. The loaded stylesheet is compiled before its CSS is nested, so parent selectors inside the caller do not affect the loaded module's rules.

## 9. Migrate configuration from global variables

Legacy Sass often configured a theme by assigning global variables before importing it:

~~~scss
// _theme.scss
$text-color: #263238 !default;
$surface-color: white !default;

// app.scss
$text-color: white;
$surface-color: #20242a;

@import "theme";
@import "components/card";
~~~

With the module system, configure the theme when it is first loaded. Components should use the theme module explicitly.

~~~scss
// _theme.scss
$text-color: #263238 !default;
$surface-color: white !default;

// components/_card.scss
@use "../theme";

.card {
  color: theme.$text-color;
  background-color: theme.$surface-color;
}

// app.scss
@use "theme" with (
  $text-color: white,
  $surface-color: #20242a
);

@use "components/card";
~~~

The component loads the same theme module. It sees the configuration applied by the entrypoint. Configure a module on its first load; a later @use cannot reconfigure a module that is already loaded.

## 10. Use the Sass Migrator

The Sass Migrator can update Sass imports and add module namespaces. Install it as a project development tool:

~~~bash
npm install --save-dev sass-migrator
~~~

Preview a module migration before changing files:

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Review the proposed changes, then apply the migration:

~~~bash
npx sass-migrator module --migrate-deps src/scss/app.scss
~~~

The module migrator can update dependencies and member references. It is not just a text replacement from @import to @use.

If the entrypoint uses load paths, pass matching --load-path options to the migrator so it can resolve dependencies:

~~~bash
npx sass-migrator --load-path=src/scss module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

The migrator generally treats files resolved from load paths as dependencies, so it does not rewrite third-party code unless you deliberately manage that code yourself.

## 11. Migrate built-in calls without migrating imports

If you want to update global built-in calls first while keeping Sass @import temporarily, use the built-in-only option:

~~~bash
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

This updates global built-in calls to module-based calls while leaving @import rules in place. It can be useful as a smaller migration step, but it does not remove the @import deprecation.

## 12. Review migrated output and behavior

After migration, review more than whether the command exits successfully.

Check:

- Every Sass dependency is loaded from a clear module URL.
- Variables, functions, and mixins use the correct namespace.
- The same module is not configured after it has already been loaded.
- CSS order and cascade behavior are still intended.
- Repeated Sass imports did not previously emit duplicate CSS that the module system now emits once.
- Nested imports were converted to a suitable mixin or meta.load-css() pattern.
- Public library members remain available through the documented entrypoint.
- Plain CSS @import rules remain browser imports.
- Compiler output is visually and behaviorally acceptable for the application.

The migration changes how Sass dependencies and CSS emission work, so review the diff and generated CSS in the project.

## 13. Handle third-party warnings carefully

A dependency may still use Sass @import or global built-ins. Prefer updating to a supported dependency release or contacting its maintainer.

Dart Sass can quiet dependency warnings in a CLI build with --quiet-deps when those dependencies are resolved through load paths. This option does not fix the deprecated code; it only reduces warning output from dependencies.

~~~bash
npx sass --quiet-deps src/scss/app.scss dist/app.css
~~~

Do not use warning suppression to hide deprecations in application code. For a temporary, narrowly scoped compatibility need, Sass has specific deprecation silencing options. The @import and global-builtin deprecations use separate identifiers, so silencing one does not silence the other.

## 14. Common migration mistakes

### Replacing every @import with @use

Some imports are plain CSS imports intended for the browser. Identify Sass module imports before changing them.

### Moving @use inside a media query

@use must appear at the top level. For nested CSS, use a mixin or sass:meta's load-css() as appropriate.

### Updating imports but leaving global built-ins

@use migration does not mean every function call was correctly namespaced. Search for global Sass built-ins and review their deprecation warnings.

### Adding namespaces without checking collisions

Two modules may have the same default namespace. Choose clear aliases so all references are unambiguous.

### Running the migrator on only one file

If related files depend on the old global scope, migrate dependencies or pass all relevant stylesheets. The module migrator needs to read dependency files even if it will not edit them.

### Migrating dependencies from load paths unintentionally

Load-path files are treated as dependencies by the migrator. Review which files are under your control and which are third-party code.

### Suppressing warnings instead of fixing owned code

Silencing warnings can be a temporary compatibility measure, but it does not update the Sass source or prepare it for future releases.

## 15. Summary

- Sass @import and global built-in function calls were deprecated in Dart Sass 1.80.0.
- @use and @forward provide explicit module loading and controlled public APIs.
- Use namespaces such as map.get(), list.nth(), and math.unit() for Sass built-ins.
- Plain CSS @import rules are separate and may remain browser-managed imports.
- @use must be top-level; use a mixin or meta.load-css() for formerly nested Sass imports.
- Configure theme modules on their first @use.
- Preview Sass Migrator changes, include dependencies when appropriate, and review the diff and generated CSS.
- Treat third-party warnings separately and avoid hiding deprecations in code you own.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="033-emitted-css-and-entrypoints.md">033 - Generated CSS and Entry Points</a></td><td><a href="../qnas/034-legacy-import-and-global-built-in-deprecations.md">Q&amp;A</a></td><td><a href="../examples/034-legacy-import-and-global-built-in-deprecations.md">Example</a></td><td><a href="035-migrating-to-sass-modules.md">035 - Migrating to Sass Modules</a></td></tr>
  </tbody>
</table>
