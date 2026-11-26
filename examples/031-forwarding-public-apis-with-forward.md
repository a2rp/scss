# 031 — Forwarding Public APIs with @forward: Examples

These examples use a small Sass library split across partials. Each section shows the source files, the public entrypoint, and how a consuming stylesheet uses that API.

> Run the examples from the directory shown in each command. Sass resolves module URLs relative to the stylesheet that contains the rule.

## 1. Create one public entrypoint

### File layout

~~~text
project/
├── src/
│   ├── app.scss
│   └── design-system/
│       ├── _index.scss
│       ├── _tokens.scss
│       └── _buttons.scss
└── dist/
~~~

### Token module

~~~scss
// src/design-system/_tokens.scss
$brand-color: #1769aa;
$text-color: #263238;
$space-2: 0.5rem;
~~~

### Button module

~~~scss
// src/design-system/_buttons.scss
@mixin button-reset {
  appearance: none;
  border: 0;
  font: inherit;
}

@mixin primary-button {
  @include button-reset;
  padding: 0.6rem 1rem;
  border-radius: 0.25rem;
  background-color: #1769aa;
  color: white;
  cursor: pointer;
}
~~~

### Public entrypoint

~~~scss
// src/design-system/_index.scss
@forward "tokens";
@forward "buttons";
~~~

### Consumer

~~~scss
// src/app.scss
@use "design-system" as ui;

.button {
  @include ui.primary-button;
}

.link {
  color: ui.$brand-color;
}
~~~

Compile it:

~~~bash
npx sass src/app.scss dist/app.css
~~~

The consumer refers only to the library entrypoint and its chosen namespace. Internal partial filenames can be reorganized while the public names stay stable.

## 2. Publish an allow-list with show

Use show when the entrypoint should expose only a few members from a larger module.

~~~scss
// src/design-system/_buttons.scss
@mixin button-reset {
  border: 0;
  font: inherit;
}

@mixin primary-button {
  @include button-reset;
  padding: 0.6rem 1rem;
  background: #1769aa;
  color: white;
}

@mixin internal-focus-ring {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}

// src/design-system/_index.scss
@forward "buttons" show primary-button;
~~~

Consumer:

~~~scss
@use "design-system" as ui;

.button {
  @include ui.primary-button;
}

// This is intentionally unavailable through ui:
// @include ui.button-reset;
// @include ui.internal-focus-ring;
~~~

Expected CSS:

~~~css
.button {
  border: 0;
  font: inherit;
  padding: 0.6rem 1rem;
  background: #1769aa;
  color: white;
}
~~~

The internal mixin can still be used by another mixin in its own module. show limits what consumers can name through this entrypoint; it does not delete or rename source members.

## 3. Exclude a legacy token with hide

Use hide when most public members should remain available and only a small number should be excluded.

~~~scss
// src/design-system/_tokens.scss
$brand-color: #1769aa;
$text-color: #263238;
$surface-color: #ffffff;
$old-brand-color: #0b4f7a;

// src/design-system/_index.scss
@forward "tokens" hide $old-brand-color;
~~~

~~~scss
// src/app.scss
@use "design-system" as ui;

.notice {
  color: ui.$text-color;
  background-color: ui.$surface-color;
  border-color: ui.$brand-color;
}
~~~

The three supported tokens are available as ui members. The legacy token is omitted from this API. Variables in a show or hide list include the dollar sign.

## 4. Add different prefixes to overlapping names

Two modules can use the same internal mixin name. Prefix their forwarded names so consumers can distinguish them.

~~~scss
// src/design-system/_buttons.scss
@mixin reset {
  border: 0;
  font: inherit;
}

// src/design-system/_forms.scss
@mixin reset {
  padding: 0.5rem;
  border: 1px solid #8a94a6;
}

// src/design-system/_index.scss
@forward "buttons" as button-*;
@forward "forms" as form-*;
~~~

~~~scss
// src/app.scss
@use "design-system" as ui;

.button {
  @include ui.button-reset;
}

.field {
  @include ui.form-reset;
}
~~~

Expected CSS:

~~~css
.button {
  border: 0;
  font: inherit;
}

.field {
  padding: 0.5rem;
  border: 1px solid #8a94a6;
}
~~~

The prefix changes the names exposed by the entrypoint. The source modules still define a mixin named reset.

## 5. Forward a module and use it locally

The entrypoint can expose a module to consumers and also use its members in its own styles. Add both rules, with @forward first.

~~~scss
// src/design-system/_focus.scss
@mixin accessible-focus {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}

// src/design-system/_index.scss
@forward "focus";
@use "focus";

.button:focus-visible {
  @include focus.accessible-focus;
}
~~~

~~~scss
// src/app.scss
@use "design-system" as ui;

.link:focus-visible {
  @include ui.accessible-focus;
}
~~~

The @use gives _index.scss the focus namespace for its own selector. The @forward exposes accessible-focus through ui for the consumer. Forwarding first leaves room for downstream configuration before the local use loads the same module.

## 6. Set a library default that consumers can override

A library can provide an opinionated brand color while allowing applications to configure it.

~~~scss
// src/design-system/_theme.scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;

@mixin themed-card {
  padding: 1rem;
  color: white;
  background-color: $brand-color;
}
~~~

~~~scss
// src/design-system/_index.scss
@forward "theme" with (
  $brand-color: #4b2e83 !default
);
~~~

~~~scss
// src/app.scss
@use "design-system" as ui with (
  $brand-color: #8b3a3a
);

.card {
  @include ui.themed-card;
}
~~~

Expected CSS:

~~~css
.card {
  padding: 1rem;
  color: white;
  background-color: #8b3a3a;
}
~~~

There are two defaults: the theme module's own default and the library entrypoint's chosen default. The !default in the forwarding configuration lets the consuming stylesheet replace the library default. Module configuration must happen when that module is first loaded in a compilation.

## 7. Notice CSS emitted by a forwarded module

Forwarding loads the module, so top-level CSS in that module becomes part of the output.

~~~scss
// src/design-system/_normalize.scss
* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

// src/design-system/_index.scss
@forward "normalize";
~~~

~~~scss
// src/app.scss
@use "design-system";
~~~

Expected CSS:

~~~css
* {
  box-sizing: border-box;
}

body {
  margin: 0;
}
~~~

Treat top-level CSS as part of the entrypoint's behavior. Keep reusable declarations in mixins or functions when consumers should opt in instead of receiving CSS automatically.

## 8. Use an internal helper without publishing it

A library can use helper modules internally while forwarding only the supported public API.

~~~text
src/design-system/
├── _index.scss
├── _tokens.scss
├── _buttons.scss
├── _forms.scss
└── internal/
    └── _focus-ring.scss
~~~

~~~scss
// src/design-system/_tokens.scss
$brand-color: #1769aa;

// src/design-system/internal/_focus-ring.scss
@mixin apply($color) {
  outline: 3px solid $color;
  outline-offset: 2px;
}

// src/design-system/_buttons.scss
@use "tokens";
@use "internal/focus-ring" as focus-ring;

@mixin primary-button {
  padding: 0.6rem 1rem;
  background-color: tokens.$brand-color;
  color: white;

  &:focus-visible {
    @include focus-ring.apply(tokens.$brand-color);
  }
}

// src/design-system/_forms.scss
@mixin field {
  display: block;
  width: 100%;
  padding: 0.5rem;
}

// src/design-system/_index.scss
@forward "tokens";
@forward "buttons" show primary-button;
@forward "forms";
~~~

Consumer:

~~~scss
@use "design-system" as ui;

.button {
  @include ui.primary-button;
}
~~~

The public entrypoint exposes primary-button, tokens, and form members. The consumer cannot call the helper through ui because the internal module is not forwarded. The button module can still use that helper in its own implementation.

## 9. Build a nested public API

A library can provide focused entrypoints as well as a top-level entrypoint.

~~~text
src/design-system/
├── _index.scss
├── buttons/
│   ├── _index.scss
│   └── _base.scss
└── forms/
    ├── _index.scss
    └── _fields.scss
~~~

~~~scss
// src/design-system/buttons/_base.scss
@mixin reset {
  border: 0;
  font: inherit;
}

@mixin primary {
  @include reset;
  padding: 0.6rem 1rem;
  background: #1769aa;
  color: white;
}

// src/design-system/buttons/_index.scss
@forward "base" show primary;

// src/design-system/forms/_fields.scss
@mixin field {
  display: block;
  width: 100%;
  padding: 0.5rem;
}

// src/design-system/forms/_index.scss
@forward "fields";

// src/design-system/_index.scss
@forward "buttons" as button-*;
@forward "forms" as form-*;
~~~

A focused consumer can load buttons:

~~~scss
@use "design-system/buttons" as buttons;

.action {
  @include buttons.primary;
}
~~~

A consumer of the full entrypoint can use the prefixed members:

~~~scss
@use "design-system" as ui;

.action {
  @include ui.button-primary;
}

input {
  @include ui.form-field;
}
~~~

Choose entrypoints based on the supported API you want to maintain. A nested entrypoint should be documented just like the top-level one.

## 10. A compact library entrypoint with curated members

This final pattern combines visibility rules and prefixes without exposing implementation names.

~~~scss
// src/design-system/_tokens.scss
$brand-color: #1769aa;
$grid-unit: 0.25rem;

// src/design-system/_buttons.scss
@mixin reset {
  border: 0;
  font: inherit;
}

@mixin primary {
  @include reset;
  padding: 0.75rem 1rem;
  background: #1769aa;
  color: white;
}

// src/design-system/_forms.scss
@mixin reset {
  padding: 0.5rem;
  border: 1px solid #8a94a6;
}

// src/design-system/_index.scss
@forward "tokens" hide $grid-unit;
@forward "buttons" as button-* show primary;
@forward "forms" as form-* show reset;
~~~

~~~scss
// src/app.scss
@use "design-system" as ui;

.save-button {
  @include ui.button-primary;
}

.search-input {
  @include ui.form-reset;
  border-radius: 0.25rem;
}
~~~

The application gets a small documented API: ui.$brand-color, ui.button-primary, and ui.form-reset. The token implementation detail and unlisted button members remain outside this entrypoint's API.

## Common mistakes to check

- Using a forwarded member locally inside _index.scss without also adding @use.
- Putting @use before @forward when both load the same configurable module.
- Listing a variable in show or hide without its dollar sign.
- Assuming as button-* renames the mixin in its source file.
- Forwarding a module with top-level selectors without realizing it emits CSS.
- Treating private files or member names as public just because they exist in the repository.
- Exposing internal helpers and later finding that consumers depend on them.

## Practice tasks

1. Add a _spacing.scss module and expose only the spacing variables intended for consumers.
2. Give two modules a mixin with the same name, then add prefixes in the public entrypoint.
3. Add a library default for a configurable theme value and override it from an app entry stylesheet.
4. Add one top-level CSS rule to a forwarded module and identify where it appears in the compiled output.
5. Remove one forwarded member and describe which consumers would need a migration.

## Quick reference

~~~scss
@forward "module";
@forward "module" show mixin-name, function-name, $variable-name;
@forward "module" hide $internal-name;
@forward "module" as button-*;

@forward "theme" with (
  $brand-color: #4b2e83 !default
);

@forward "focus";
@use "focus";
~~~

The public entrypoint decides which members and names downstream users receive. Keep the API small, deliberate, and documented.

## Related material

- [Lesson: Forwarding public APIs with @forward](../lessons/031-forwarding-public-apis-with-forward.md)
- [Q&A: Forwarding public APIs with @forward](../qnas/031-forwarding-public-apis-with-forward.md)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
