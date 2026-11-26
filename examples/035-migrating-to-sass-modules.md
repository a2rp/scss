# 035 — Migrating to Sass Modules: Examples

These examples show a migration in small steps: make dependencies explicit, publish a controlled API, preserve configuration, and verify the source and compiled CSS.

## 1. Start with an implicit global dependency

### Before

~~~scss
// _tokens.scss
$brand: #1769aa;
$text: #263238;

// _button.scss
.button {
  color: $text;
  background: $brand;
}

// app.scss
@import "tokens";
@import "button";
~~~

The button partial depends on values that are only present because the entrypoint imports tokens first.

### After

~~~scss
// _tokens.scss
$brand: #1769aa;
$text: #263238;

// _button.scss
@use "tokens";

.button {
  color: tokens.$text;
  background: tokens.$brand;
}

// app.scss
@use "button";
~~~

Now the file that uses the tokens declares that dependency directly.

## 2. Give several modules their own dependencies

Suppose buttons and cards both use the same token module:

~~~scss
// _button.scss
@use "tokens";

.button {
  color: tokens.$text;
  background: tokens.$brand;
}

// _card.scss
@use "tokens";

.card {
  color: tokens.$text;
  border-color: tokens.$brand;
}

// app.scss
@use "button";
@use "card";
~~~

Each module uses its own explicit namespace. The entrypoint does not need to load tokens unless it also refers to token members itself.

## 3. Build a public facade

A library can keep implementation files internal and forward a small, supported surface.

~~~scss
// styles/_tokens.scss
$brand: #1769aa !default;
$text: #263238 !default;
$internal-shadow: 0 1px 3px rgb(0 0 0 / 15%);

// styles/_button.scss
@use "tokens";

@mixin button {
  color: tokens.$text;
  background: tokens.$brand;
}

// styles/_index.scss
@forward "tokens" show $brand, $text;
@forward "button" show button;
~~~

A consumer uses the facade instead of depending on the internal file layout:

~~~scss
@use "styles" as ui;

.primary-action {
  @include ui.button;
  border-color: ui.$brand;
}
~~~

The consumer can use the documented members through one namespace. It cannot access `$internal-shadow` because the facade does not forward it.

## 4. Add a prefix to a forwarded API

When simple member names need more context, give them a prefix at the facade:

~~~scss
// styles/_reset.scss
@mixin list {
  margin: 0;
  padding: 0;
  list-style: none;
}

// styles/_index.scss
@forward "reset" as reset-*;
~~~

~~~scss
// app.scss
@use "styles" as ui;

.navigation ul {
  @include ui.reset-list;
}
~~~

The Sass module namespace is `ui` and the forwarded member name is `reset-list`. These are separate naming layers.

## 5. Use show and hide to limit the public API

~~~scss
// styles/_colors.scss
$brand: #1769aa;
$text: #263238;
$debug-magenta: magenta;

// styles/_index.scss
@forward "colors" show $brand, $text;
~~~

Only `$brand` and `$text` are available through the facade. Alternatively, hide a known implementation detail while forwarding the other public members:

~~~scss
@forward "colors" hide $debug-magenta;
~~~

For a long-lived library, an explicit `show` list makes the intended API easy to audit when new internal members are added.

## 6. Use a facade internally and externally

A facade that uses a forwarded module internally can declare both relationships:

~~~scss
// styles/_tokens.scss
$brand: #1769aa;

// styles/_index.scss
@forward "tokens" show $brand;
@use "tokens";

@mixin outline {
  outline: 2px solid tokens.$brand;
}
~~~

~~~scss
// app.scss
@use "styles" as ui;

.link:focus-visible {
  @include ui.outline;
  color: ui.$brand;
}
~~~

Put the forward before the use when the facade should forward module configuration from its consumer.

## 7. Configure a theme before loading components

### Theme module

~~~scss
// styles/_theme.scss
$brand: #1769aa !default;
$surface: white !default;
$text: #263238 !default;
~~~

### Component module

~~~scss
// styles/_panel.scss
@use "theme";

.panel {
  color: theme.$text;
  background-color: theme.$surface;
  border-top: 4px solid theme.$brand;
}
~~~

### Dark entrypoint

~~~scss
// dark.scss
@use "styles/theme" with (
  $brand: #8ab4f8,
  $surface: #20242a,
  $text: #f5f7fa
);

@use "styles/panel";
~~~

Expected CSS:

~~~css
.panel {
  color: #f5f7fa;
  background-color: #20242a;
  border-top: 4px solid #8ab4f8;
}
~~~

The component loads the same theme module and sees the configuration established by the entrypoint.

## 8. Configure through a package facade

A public facade can forward configurable variables from the theme module:

~~~scss
// styles/_theme.scss
$brand: #1769aa !default;
$surface: white !default;

// styles/_panel.scss
@use "theme";

.panel {
  background: theme.$surface;
  border-color: theme.$brand;
}

// styles/_index.scss
@forward "theme";
@forward "panel";
~~~

A consumer can configure the public facade before it loads the panel:

~~~scss
// app.scss
@use "styles" as ui with (
  $brand: #6b4eff,
  $surface: #f6f3ff
);
~~~

Expected CSS:

~~~css
.panel {
  background: #f6f3ff;
  border-color: #6b4eff;
}
~~~

Keep forwarding order and configuration flow clear. Compile a consumer fixture to confirm the public facade behaves as intended.

## 9. Compile two theme entrypoints separately

Each entrypoint can choose its own theme configuration when compiled as its own Sass compilation:

~~~scss
// light.scss
@use "styles" with (
  $brand: #1769aa,
  $surface: white
);

@use "styles/panel";
~~~

~~~scss
// dark.scss
@use "styles" with (
  $brand: #8ab4f8,
  $surface: #20242a
);

@use "styles/panel";
~~~

Build both entrypoints independently:

~~~bash
npx sass src/scss/light.scss dist/light.css
npx sass src/scss/dark.scss dist/dark.css
~~~

Configuration is per compilation. Do not try to configure the same module twice inside one compilation; create a separate entrypoint when a separate compiled theme is needed.

## 10. Preview the migrator output

Install the migrator as a project development dependency:

~~~bash
npm install --save-dev sass-migrator
~~~

Preview the migration of an entrypoint and its dependencies:

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Review the proposed edits. Check that:

- The changed files belong to the project or are intentionally maintained.
- Namespaces are understandable.
- Each module can resolve its dependencies.
- Public Sass names are still available where promised.
- Nested imports receive a deliberate replacement.
- Configuration is applied in the intended entrypoint.

Apply the reviewed migration:

~~~bash
npx sass-migrator module --migrate-deps src/scss/app.scss
~~~

Then compile using the project's normal Sass build command.

## 11. Include a load path in the migration preview

If the project resolves modules from a shared directory, include that load path:

~~~bash
npx sass-migrator --load-path=src/scss module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Pass the same resolution settings used by the build. Confirm whether the files reached through that path are maintained by your team before applying edits.

## 12. Migrate built-ins as a separate change

When a project needs to separate the two deprecation fixes, migrate built-ins first:

~~~bash
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

For example, a global call such as `map-get($colors, "brand")` can become a namespaced call using `sass:map`. Sass `@import` rules remain for a later migration. Review the previous chapter for built-in module examples.

## 13. Replace a nested import with an opt-in mixin

### Before

~~~scss
// _print.scss
.print-only {
  display: none;
}

// app.scss
@media print {
  @import "print";
}
~~~

### After

~~~scss
// _print.scss
@mixin styles {
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
  .print-only {
    display: block !important;
  }
}
~~~

The module is loaded at the top level. Its mixin emits the styles only where the entrypoint includes it.

## 14. Preserve a browser-managed CSS import

~~~scss
// app.scss
@import url("https://example.test/fonts.css");
@import "legacy-theme.css";

.page-title {
  font-family: sans-serif;
}
~~~

These CSS imports remain in the output for the browser:

~~~css
@import url("https://example.test/fonts.css");
@import "legacy-theme.css";

.page-title {
  font-family: sans-serif;
}
~~~

Do not turn these into Sass module loads. Migrate Sass imports that expose Sass members; preserve browser-managed CSS imports.

## 15. Compile a consumer fixture for a library facade

Create a small fixture that uses only the documented API:

~~~scss
// test/consumer.scss
@use "../src/scss/styles" as ui;

.demo-button {
  @include ui.button;
  color: ui.$text;
}
~~~

Compile it:

~~~bash
npx sass test/consumer.scss test/consumer.css
~~~

Review that the documented members resolve and that the expected CSS is emitted. If an old API remains supported during transition, add another fixture for the compatibility entrypoint and compile both fixtures.

## 16. Compare generated CSS before and after

Compile the same entrypoint with the same options into two locations:

~~~bash
npx sass --style=expanded src/scss-before/app.scss /tmp/app-before.css
npx sass --style=expanded src/scss-after/app.scss /tmp/app-after.css
~~~

Compare the files using a diff tool:

~~~bash
diff -u /tmp/app-before.css /tmp/app-after.css
~~~

Investigate differences instead of accepting them automatically. Some changes may be intended, such as removing duplicated output caused by repeated imports. Others may signal a missing mixin include, a changed configuration value, or a rule emitted in a different order.

## 17. Keep a migration ledger

For a large project, track each slice so warnings and ownership do not get lost:

| Area | Status | Owner | Verification |
| --- | --- | --- | --- |
| Token modules | Converted | App team | App bundle matches |
| Button and card partials | Converted | App team | Consumer fixture compiles |
| Vendor theme package | Waiting for upgrade | Third party | Warning tracked |
| Nested print styles | Converted to mixin | App team | Print CSS reviewed |
| Public facade | Published | Library team | API fixture compiles |

Update the ledger as files are migrated. Do not mark a dependency complete just because its warning was hidden.

## 18. Practice tasks

1. Convert a small Sass import chain to explicit `@use` rules.
2. Create a facade that exposes only selected variables and mixins.
3. Add a prefix to one forwarded mixin and consume it from an entrypoint.
4. Configure a theme from an entrypoint before loading a dependent component.
5. Preview a migrator run and list every changed file that is not project-owned.
6. Replace a nested import with a mixin and confirm its CSS remains in the intended at-rule.
7. Compile a library consumer fixture and compare representative CSS before and after migration.

## Quick reference

~~~scss
@use "tokens" as theme;
@forward "tokens" show $brand, $text;
@forward "mixins" as utility-*;

@use "theme" with (
  $brand: #8ab4f8
);
~~~

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
npx sass-migrator module --migrate-deps src/scss/app.scss
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
npx sass src/scss/app.scss dist/app.css
~~~

## Related material

- [Lesson: Migrating to Sass modules](../lessons/035-migrating-to-sass-modules.md)
- [Q&A: Migrating to Sass modules](../qnas/035-migrating-to-sass-modules.md)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass Migrator](https://sass-lang.com/documentation/cli/migrator/)
- [Sass import and global built-in deprecations](https://sass-lang.com/documentation/breaking-changes/import/)
