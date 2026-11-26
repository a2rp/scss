# 033 - Generated CSS and Entry Points: Examples

These examples focus on what Sass writes to CSS. The entrypoint is the source file passed to Sass; modules loaded through it contribute their CSS to that compilation.

## 1. Compile a basic entrypoint

### Files

~~~text
project/
├── src/
│   └── scss/
│       ├── app.scss
│       ├── _base.scss
│       ├── _tokens.scss
│       └── components/
│           └── _card.scss
└── dist/
~~~

~~~scss
// src/scss/_base.scss
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}

// src/scss/_tokens.scss
$text-color: #263238;
$surface-color: #ffffff;
$space-medium: 1rem;

// src/scss/components/_card.scss
@use "../tokens";

.card {
  padding: tokens.$space-medium;
  color: tokens.$text-color;
  background-color: tokens.$surface-color;
}

// src/scss/app.scss
@use "base";
@use "components/card";
~~~

Compile app.scss:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

Expected CSS:

~~~css
body {
  margin: 0;
  font-family: system-ui, sans-serif;
}

.card {
  padding: 1rem;
  color: #263238;
  background-color: #ffffff;
}
~~~

The entrypoint loads the base and card modules. The token variables help produce declarations but do not create a selector by themselves.

## 2. A mixin definition is quiet until it is included

~~~scss
// src/scss/_button.scss
@mixin primary {
  display: inline-flex;
  padding: 0.6rem 1rem;
  border: 0;
  color: white;
  background-color: #1769aa;
}
~~~

Loading the module without including the mixin emits no button rule:

~~~scss
// src/scss/app.scss
@use "button";
~~~

Now include the mixin inside a selector:

~~~scss
// src/scss/app.scss
@use "button";

.button {
  @include button.primary;
}
~~~

Expected CSS:

~~~css
.button {
  display: inline-flex;
  padding: 0.6rem 1rem;
  border: 0;
  color: white;
  background-color: #1769aa;
}
~~~

The declarations appear at the include site. This lets a module provide reusable styling without forcing a selector into every application that loads it.

## 3. CSS from a transitive dependency is emitted once

### Files

~~~text
src/scss/
├── app.scss
├── _base.scss
├── _header.scss
└── _footer.scss
~~~

~~~scss
// src/scss/_base.scss
:root {
  font-family: system-ui, sans-serif;
}

// src/scss/_header.scss
@use "base";

.site-header {
  padding: 1rem;
}

// src/scss/_footer.scss
@use "base";

.site-footer {
  padding: 1rem;
}

// src/scss/app.scss
@use "header";
@use "footer";
~~~

Expected CSS:

~~~css
:root {
  font-family: system-ui, sans-serif;
}

.site-header {
  padding: 1rem;
}

.site-footer {
  padding: 1rem;
}
~~~

Both modules depend on base, but the same base module is evaluated once in this compilation.

## 4. Different modules can emit repeated selectors

~~~scss
// src/scss/_header.scss
.notice {
  color: #263238;
}

// src/scss/_footer.scss
.notice {
  color: #263238;
}

// src/scss/app.scss
@use "header";
@use "footer";
~~~

Expected CSS contains both rules:

~~~css
.notice {
  color: #263238;
}

.notice {
  color: #263238;
}
~~~

The rules come from two different source modules. Sass does not deduplicate them. If the repetition is unintended, remove or consolidate the original source rules.

## 5. Source order affects the cascade

~~~scss
// src/scss/_base.scss
.notice {
  color: #263238;
  border-color: #d5dbe3;
}

// src/scss/_theme.scss
.notice {
  color: #1769aa;
}

// src/scss/app.scss
@use "base";
@use "theme";
~~~

Generated CSS:

~~~css
.notice {
  color: #263238;
  border-color: #d5dbe3;
}

.notice {
  color: #1769aa;
}
~~~

The selectors have the same specificity. The later color declaration from theme wins, while the base border-color remains in effect.

Changing the order changes which color wins:

~~~scss
// src/scss/app.scss
@use "theme";
@use "base";
~~~

Keep entrypoint order explicit when modules emit overlapping CSS.

## 6. A public entrypoint can emit CSS and expose mixins

~~~text
src/scss/
├── app.scss
└── design-system/
    ├── _index.scss
    ├── _normalize.scss
    └── _buttons.scss
~~~

~~~scss
// src/scss/design-system/_normalize.scss
* {
  box-sizing: border-box;
}

// src/scss/design-system/_buttons.scss
@mixin primary {
  padding: 0.6rem 1rem;
  border: 0;
  color: white;
  background-color: #1769aa;
}

// src/scss/design-system/_index.scss
@forward "normalize";
@forward "buttons";

// src/scss/app.scss
@use "design-system" as ui;

.button {
  @include ui.primary;
}
~~~

Expected CSS:

~~~css
* {
  box-sizing: border-box;
}

.button {
  padding: 0.6rem 1rem;
  border: 0;
  color: white;
  background-color: #1769aa;
}
~~~

Loading the public API loads the normalize module, so its top-level CSS is emitted. The button declarations appear at the @include site.

## 7. Make component CSS opt-in with a mixin

### Component module

~~~scss
// src/scss/components/_dialog.scss
@mixin styles {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background-color: rgb(0 0 0 / 0.5);
}
~~~

### Entrypoint that loads but does not include it

~~~scss
// src/scss/app.scss
@use "components/dialog";
~~~

This compilation emits no dialog selector because the module only defines a mixin.

### Entrypoint that opts into the styles

~~~scss
// src/scss/app.scss
@use "components/dialog";

.dialog-backdrop {
  @include dialog.styles;
}
~~~

Expected CSS:

~~~css
.dialog-backdrop {
  position: fixed;
  inset: 0;
  display: grid;
  place-items: center;
  background-color: rgb(0 0 0 / 0.5);
}
~~~

The consuming stylesheet controls the selector where the declarations are emitted.

## 8. Compile separate application entrypoints

### Files

~~~text
src/scss/
├── app.scss
├── admin.scss
├── _reset.scss
├── _tokens.scss
└── components/
    ├── _site.scss
    └── _admin-panel.scss
~~~

~~~scss
// src/scss/_reset.scss
* {
  box-sizing: border-box;
}

// src/scss/components/_site.scss
.site-header {
  padding: 1rem;
}

// src/scss/components/_admin-panel.scss
.admin-panel {
  padding: 1rem;
}

// src/scss/app.scss
@use "reset";
@use "components/site";

// src/scss/admin.scss
@use "reset";
@use "components/admin-panel";
~~~

Compile each output:

~~~bash
npx sass src/scss/app.scss dist/css/app.css
npx sass src/scss/admin.scss dist/css/admin.css
~~~

The app.css output includes the reset and site header styles. The admin.css output includes the reset and admin panel styles. The reset can appear in both files because Sass compiled each entrypoint separately.

A page should load only the CSS bundle or bundles it needs.

## 9. Compile a directory of entrypoints

~~~text
src/scss/
├── app.scss
├── admin.scss
├── _reset.scss
└── components/
    ├── _site.scss
    └── _admin-panel.scss
~~~

~~~bash
npx sass src/scss:dist/css
~~~

Dart Sass creates CSS outputs for non-partial Sass files in the source directory tree, such as app.css and admin.css. The underscore-prefixed modules are not emitted as standalone files, but the entrypoints can load them.

Use directory mode when every non-partial Sass file in the directory is intended to compile. Otherwise, compile explicit entrypoints.

## 10. Choose an output style

Dart Sass's default expanded style is readable:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

Example output:

~~~css
.card {
  padding: 1rem;
  color: #263238;
}
~~~

Compressed style removes formatting whitespace:

~~~bash
npx sass --style=compressed src/scss/app.scss dist/app.min.css
~~~

Example output:

~~~css
.card{padding:1rem;color:#263238}
~~~

The Sass source and declarations are the same. Only the output formatting changes.

## 11. Inspect source maps

When Sass writes CSS, it emits a source map by default:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

The output can include:

~~~text
dist/
├── app.css
└── app.css.map
~~~

To disable the source map:

~~~bash
npx sass --no-source-map src/scss/app.scss dist/app.css
~~~

Source maps help developer tools connect generated CSS to Sass source files. They do not add selectors or declarations to app.css.

## 12. Trace an unexpected rule

Suppose an unexpected global selector appears in dist/app.css:

~~~css
html {
  font-family: system-ui, sans-serif;
}
~~~

Trace it from the entrypoint:

~~~scss
// src/scss/app.scss
@use "components/card";
~~~

Then inspect the component's dependencies:

~~~scss
// src/scss/components/_card.scss
@use "../base";
~~~

Finally inspect _base.scss for top-level CSS:

~~~scss
// src/scss/_base.scss
html {
  font-family: system-ui, sans-serif;
}
~~~

The rule comes from a transitive dependency. If it should be global, keep that dependency intentional. If it should be optional, move the declarations into a mixin and include them explicitly.

## 13. Avoid outputting component CSS unintentionally

Unintended top-level selector:

~~~scss
// src/scss/components/_tooltip.scss
.tooltip {
  position: absolute;
  padding: 0.5rem;
}
~~~

Any entrypoint that loads this module receives the .tooltip rule, even if the page never renders a tooltip.

Opt-in version:

~~~scss
// src/scss/components/_tooltip.scss
@mixin styles {
  position: absolute;
  padding: 0.5rem;
}
~~~

~~~scss
// src/scss/app.scss
@use "components/tooltip";

.tooltip {
  @include tooltip.styles;
}
~~~

Now the consuming stylesheet chooses if and where the tooltip declarations appear.

## Practice tasks

1. Create a reset module with a top-level rule and confirm it appears when loaded.
2. Create a mixin-only module and compare output before and after including the mixin.
3. Load one CSS-emitting module through two dependencies and observe that its CSS appears once.
4. Add identical selectors to two different modules and observe that both rules remain.
5. Create app.scss and admin.scss and compare their independently compiled outputs.
6. Compile a directory containing two entrypoints and several partials.
7. Generate compressed CSS, then compare its declarations with expanded output.
8. Use a source map to locate the Sass source for a generated selector.

## Quick reference

~~~scss
// Top-level CSS is emitted when this module is loaded.
@use "base";

// Mixin CSS is emitted when included.
@use "components/button";

.button {
  @include button.primary;
}
~~~

~~~bash
npx sass src/scss/app.scss dist/css/app.css
npx sass --style=compressed src/scss/app.scss dist/css/app.css
npx sass --no-source-map src/scss/app.scss dist/css/app.css
npx sass src/scss:dist/css
~~~

## Related material

- [Lesson: Generated CSS and entry points](../lessons/033-emitted-css-and-entrypoints.md)
- [Q&A: Generated CSS and entry points](../qnas/033-emitted-css-and-entrypoints.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="032-partials-load-paths-and-file-naming.md">032 - Partials, Load Paths, and File Naming: Examples</a></td><td><a href="../lessons/033-emitted-css-and-entrypoints.md">Lesson</a></td><td><a href="../qnas/033-emitted-css-and-entrypoints.md">Q&amp;A</a></td><td><a href="034-legacy-import-and-global-built-in-deprecations.md">034 - Legacy @import and Global Built-In Deprecations: Examples</a></td></tr>
  </tbody>
</table>
