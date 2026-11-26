# Component styles and architecture - Examples

These examples use Dart Sass and its module system. Treat each numbered example as a separate project or snippet unless it says it continues another section. In a real project, compile the application entry point rather than compiling every partial.

Example command:

~~~sh
sass styles/app.scss public/app.css
~~~

## 1. Start with a responsibility-based structure

~~~text
styles/
  app.scss
  settings/
    _tokens.scss
    _index.scss
  foundations/
    _reset.scss
    _typography.scss
  components/
    _button.scss
    _card.scss
  utilities/
    _visually-hidden.scss
~~~

Keep the initial structure small. Add directories when they clarify ownership or responsibility.

## 2. Compose an application entry point

~~~scss
// app.scss
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "components/card";
@use "utilities/visually-hidden";
~~~

This entry point declares which modules contribute to the application stylesheet. The order is visible in one place.

## 3. Define value-only tokens

~~~scss
// settings/_tokens.scss
$color-brand: #3157d5;
$color-text: #182033;
$color-surface: #ffffff;
$radius-control: 0.375rem;
$space-medium: 1rem;
~~~

This module defines Sass values and emits no CSS rules by itself.

## 4. Use tokens with an explicit namespace

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  border: 0;
  border-radius: tokens.$radius-control;
  padding: tokens.$space-medium;
  background: tokens.$color-brand;
  color: tokens.$color-surface;
}
~~~

The namespace makes the dependency clear. A file that needs tokens should load them directly.

## 5. Forward a small settings API

~~~scss
// settings/_index.scss
@forward "tokens";
~~~

~~~scss
// components/_card.scss
@use "../settings" as settings;

.card {
  border: 1px solid #d6dce8;
  border-radius: settings.$radius-control;
  padding: settings.$space-medium;
  color: settings.$color-text;
}
~~~

The index file presents the settings module through one path. Add `show` or `hide` clauses when only some members should be public.

## 6. Keep private implementation details private

~~~scss
// settings/_tokens.scss
$_private-scale: 0.25rem;

@function _scale($step) {
  @return $_private-scale * $step;
}

$space-small: _scale(2);
$space-medium: _scale(4);
~~~

Names beginning with an underscore or hyphen are private to the module that defines them. Consumers can use the public spacing variables but cannot access the private helper.

## 7. Write a component with a clear root selector

~~~scss
// components/_card.scss
@use "../settings/tokens" as tokens;

.card {
  padding: tokens.$space-medium;
  border: 1px solid #d6dce8;
  border-radius: tokens.$radius-control;

  &__title {
    margin: 0 0 0.5rem;
    color: tokens.$color-text;
  }

  &--featured {
    border-color: tokens.$color-brand;
  }
}
~~~

The parent selector creates `.card__title` and `.card--featured`. The module defines one component and its related selectors.

## 8. Add a parameterized mixin

~~~scss
// foundations/_focus.scss
@mixin focus-ring($color: #3157d5) {
  &:focus-visible {
    outline: 3px solid $color;
    outline-offset: 2px;
  }
}
~~~

~~~scss
// components/_button.scss
@use "../foundations/focus";

.button {
  border: 0;

  @include focus.focus-ring(#2747b8);
}
~~~

A mixin is useful here because callers can choose the focus color while sharing the same rule structure.

## 9. Use a content block to wrap caller styles

~~~scss
// foundations/_media.scss
@mixin below($width) {
  @media (max-width: $width) {
    @content;
  }
}
~~~

~~~scss
// components/_card.scss
@use "../foundations/media";

.card {
  padding: 1.5rem;

  @include media.below(40rem) {
    padding: 1rem;
  }
}
~~~

The content block is emitted inside the media query. The mixin controls the wrapper; the component controls the declarations.

## 10. Return a value from a Sass function

~~~scss
// settings/_spacing.scss
@function spacing($step) {
  @return $step * 0.25rem;
}
~~~

~~~scss
// components/_panel.scss
@use "../settings/spacing";

.panel {
  padding: spacing.spacing(4);
  gap: spacing.spacing(2);
}
~~~

A function returns Sass data to the caller. It does not emit a selector or a declaration on its own.

## 11. Use a placeholder for a shared semantic relationship

~~~scss
%message-base {
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
}

.message {
  @extend %message-base;
  background: #edf1ff;
}

.message--error {
  @extend %message-base;
  color: #8c1d2c;
}
~~~

The placeholder is not output as a selector by itself. Sass creates output for selectors that extend it.

For a configurable style chunk, a mixin may be more direct. Use a placeholder when the selector relationship itself is intended.

## 12. Compare mixin reuse with extension

Mixin approach:

~~~scss
@mixin message-base {
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
}

.message {
  @include message-base;
  background: #edf1ff;
}

.message--error {
  @include message-base;
  color: #8c1d2c;
}
~~~

Placeholder approach:

~~~scss
%message-base {
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
}

.message {
  @extend %message-base;
  background: #edf1ff;
}

.message--error {
  @extend %message-base;
  color: #8c1d2c;
}
~~~

Both can be appropriate. The mixin copies declarations into include locations; the extension combines selectors. Choose the behavior that best describes the relationship and keeps compiled selectors predictable.

## 13. Keep global foundations separate

~~~scss
// foundations/_reset.scss
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
}
~~~

~~~scss
// foundations/_typography.scss
body {
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}
~~~

Global document defaults belong in a clearly named foundation module. A component module should not unexpectedly reset the entire document.

## 14. Add a small utility module

~~~scss
// utilities/_visually-hidden.scss
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
~~~

Load this module from the application entry point only if the application includes the utility class.

## 15. Assign styles to cascade layers

First create a module whose only CSS is the layer order statement:

~~~scss
// foundations/_layer-order.scss
@layer reset, foundations, components, utilities;
~~~

Then put each module’s rules in its named layer:

~~~scss
// foundations/_reset.scss
@layer reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }
}
~~~

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

@layer components {
  .button {
    border: 0;
    border-radius: tokens.$radius-control;
    background: tokens.$color-brand;
    color: tokens.$color-surface;
  }
}
~~~

Load the order module before modules that emit layered rules:

~~~scss
// app.scss
@use "foundations/layer-order";
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "utilities/visually-hidden";
~~~

This establishes the named layer order before those layer blocks are emitted. Use layers only if they help the project manage the cascade.

## 16. Configure a module at compile time

~~~scss
// settings/_theme.scss
$brand: #3157d5 !default;

.button {
  background: $brand;
}
~~~

~~~scss
// app.scss
@use "settings/theme" with (
  $brand: #2747b8
);
~~~

The configuration is applied when the module is first loaded in this compilation. It does not change after the page loads in a browser.

## 17. Build a runtime theme with CSS custom properties

~~~scss
:root {
  --surface: #ffffff;
  --text: #182033;
  --brand: #3157d5;
}

[data-theme="dark"] {
  --surface: #151a24;
  --text: #f3f5fa;
  --brand: #9aaeff;
}

.card {
  background: var(--surface);
  color: var(--text);
}

.button {
  background: var(--brand);
}
~~~

Sass can generate these declarations from maps, but CSS custom properties make the selected values available to the browser at runtime.

## 18. Load a shared dependency only once

~~~scss
// _tokens.scss
$brand: #3157d5;
~~~

~~~scss
// _button.scss
@use "tokens" as tokens;

.button {
  background: tokens.$brand;
}
~~~

~~~scss
// _link.scss
@use "tokens" as tokens;

.link {
  color: tokens.$brand;
}
~~~

~~~scss
// app.scss
@use "button";
@use "link";
~~~

Both components explicitly depend on the token module. Sass’s module system loads the shared module once in this compilation. Separate app entry points compile independently.

## 19. Avoid a hidden dependency

Hidden dependency pattern:

~~~scss
// _tokens.scss
$brand: #3157d5;
~~~

~~~scss
// _button.scss
.button {
  background: $brand;
}
~~~

The component only works if some other file happened to load the token into a global namespace. With modules, make the dependency explicit:

~~~scss
@use "tokens" as tokens;

.button {
  background: tokens.$brand;
}
~~~

## 20. Select only the components an entry point needs

~~~scss
// pages/marketing.scss
@use "../foundations/reset";
@use "../components/button";
@use "../components/card";
~~~

~~~scss
// pages/account.scss
@use "../foundations/reset";
@use "../components/button";
@use "../components/form-field";
~~~

Separate entry points can produce separate bundles. Each should include only the shared foundations and components required by that bundle.

## 21. Practice: audit a module

For a Sass module you already have, write down:

1. Which Sass members it defines and which it uses.
2. Whether loading it emits CSS.
3. Which application entry points load it.
4. Whether its public API is intentionally exposed.
5. Whether its selectors are global or scoped under a component root.
6. Whether each mixin, function, or extension has a clear reason to exist.

Then simplify one hidden dependency or overly broad selector.

## 22. Practice: create a button module

Create these files:

~~~text
styles/
  app.scss
  settings/
    _tokens.scss
  foundations/
    _focus.scss
  components/
    _button.scss
~~~

Requirements:

- Put brand and text colors in the token module.
- Put a configurable focus-ring mixin in the focus module.
- Make the button module load both dependencies with namespaces.
- Keep the button’s base and disabled selectors under `.button`.
- Load the button from the app entry point.

## 23. Reference links

## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="040-maps-for-design-systems.md">040 - Maps for design systems - Examples</a></td><td><a href="../lessons/041-component-styles-and-architecture.md">Lesson</a></td><td><a href="../qnas/041-component-styles-and-architecture.md">Q&amp;A</a></td><td><a href="042-project-structure-and-naming-conventions.md">042 - Project structure and naming conventions - Examples</a></td></tr>
  </tbody>
</table>
