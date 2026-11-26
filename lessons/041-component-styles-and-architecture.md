# Component styles and architecture

A stylesheet architecture defines where Sass code lives, how modules share values, and how component CSS reaches the final stylesheet. A good structure helps you find a rule, understand its dependencies, and change it without creating accidental global effects.

There is no single folder layout that fits every project. Start with one clear CSS entry point, organize Sass by responsibility, and make each module’s public API explicit.

## Learning goals

After this lesson, you should be able to:

- Separate the application entry point, foundations, tokens, components, and utilities.
- Use Sass modules with `@use` and `@forward` instead of relying on a global namespace.
- Control which files emit CSS and how that CSS is included.
- Design component selectors and Sass APIs with clear boundaries.
- Choose between a mixin, function, component rule, and `@extend`.
- Avoid duplicate output, hidden dependencies, deep nesting, and oversized modules.

## 1. Think in responsibilities

A practical stylesheet system often has these responsibilities:

- **Entry point:** decides which styles are included and in what order.
- **Foundations:** reset, document defaults, typography, and shared primitives.
- **Tokens:** colors, spacing, type scales, radii, and other design decisions.
- **Components:** styles for interface pieces such as buttons, cards, and forms.
- **Utilities:** small, intentionally composable classes such as visually hidden text.
- **Vendor styles:** external stylesheets, when the project needs them.

These are roles, not mandatory directories. A small project may keep a few modules together; a larger system may split them by product area.

## 2. A starting directory structure

One possible structure is:

~~~text
styles/
  app.scss
  settings/
    _tokens.scss
    _breakpoints.scss
  foundations/
    _reset.scss
    _typography.scss
  components/
    _button.scss
    _card.scss
    _form-field.scss
  utilities/
    _visually-hidden.scss
~~~

A leading underscore marks a Sass partial, which is generally intended to be loaded by another stylesheet. The underscore is omitted from the module URL, so `@use "components/button"` can load `_button.scss`.

Keep names descriptive. If a file becomes hard to navigate, split it by responsibility; avoid creating many tiny files whose only effect is extra indirection.

## 3. Keep one application entry point

The application entry point controls the assembled stylesheet:

~~~scss
// app.scss
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "components/card";
@use "components/form-field";
@use "utilities/visually-hidden";
~~~

Compile the entry point, not each partial independently. Loading a module with `@use` makes that module’s Sass members available to the current stylesheet and includes its CSS in the compiled output. A module loaded multiple times through the module system is included only once in that compilation.

A clear entry point makes the included CSS and its order easy to review. Avoid having many entry points that include overlapping component modules unless the project intentionally builds separate bundles.

## 4. Use `@use` for dependencies

Each stylesheet should declare the modules it uses:

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  background: tokens.$color-brand;
  border-radius: tokens.$radius-control;
}
~~~

Members are namespaced by default, so `tokens.$color-brand` shows where the value came from. This reduces name collisions and hidden dependencies. Every file that uses a token should load the module itself; it should not depend on another file having loaded it earlier.

Use explicit namespaces for external modules and shared project modules. `@use "sass:map";`, for example, makes map functions available under `map.`.

## 5. Expose a small API with `@forward`

A module can forward selected members from other modules through a public entry point:

~~~scss
// settings/_index.scss
@forward "tokens";
@forward "breakpoints" show $breakpoint-table;
~~~

A component can then load the settings API:

~~~scss
// components/_button.scss
@use "../settings" as settings;

.button {
  background: settings.$color-brand;
}
~~~

`@forward` makes public variables, functions, and mixins available to stylesheets that `@use` the forwarding module. It can show or hide members. Use it to define a deliberate package API, not to flatten every private implementation detail into one huge namespace.

A forwarded member is available through the forwarding module’s namespace; forwarding does not make it globally available.

## 6. Separate CSS-emitting modules from value-only modules

A token module may contain only variables and functions:

~~~scss
// settings/_tokens.scss
$color-brand: #3157d5;
$color-text: #182033;
$radius-control: 0.375rem;
~~~

A component module may contain selectors and declarations:

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  border: 0;
  border-radius: tokens.$radius-control;
  background: tokens.$color-brand;
  color: #fff;
}
~~~

This separation makes it easier to see which dependencies contribute CSS. A module may expose values and CSS together when that is its clear purpose, but avoid loading a broad bundle just to access one variable.

## 7. Make selector boundaries explicit

A component should have a recognizable root class and a limited selector scope:

~~~scss
.card {
  padding: 1rem;
  border: 1px solid #d6dce8;
  border-radius: 0.75rem;

  &__title {
    margin: 0;
    font-size: 1.25rem;
  }

  &--featured {
    border-color: #3157d5;
  }
}
~~~

The parent selector `&` represents the current selector. Here the output names are `.card__title` and `.card--featured`.

Choose a class naming convention that works for the project and document it. Keep nesting shallow enough that selectors remain visible and reusable. Avoid styling broad element names from inside a component, such as `h2` or `a`, when that rule could unintentionally affect nested content.

A component’s selector contract should be stable enough that markup and tests can rely on it, while remaining independent of the Sass file layout.

## 8. Use mixins for configurable style output

A mixin is useful when a repeated style pattern accepts arguments or emits declarations at the include site:

~~~scss
// _focus.scss
@mixin focus-ring($color: #3157d5) {
  &:focus-visible {
    outline: 3px solid $color;
    outline-offset: 2px;
  }
}
~~~

~~~scss
// components/_button.scss
@use "../settings/focus";

.button {
  border: 0;

  @include focus.focus-ring(#2747b8);
}
~~~

Mixins can include declarations, nested rules, or a content block. Keep the argument list focused and document unusual options.

Do not wrap every selector in a mixin. If a component should always be present when its module is loaded, a normal component rule is simpler than an include API.

## 9. Use functions for values

A Sass function returns a value that can be used in a declaration or another Sass expression:

~~~scss
@use "sass:map";

$spacing: ("small": 0.5rem, "large": 1.5rem);

@function space($name) {
  @if not map.has-key($spacing, $name) {
    @error "Unknown spacing token #{$name}.";
  }

  @return map.get($spacing, $name);
}

.panel {
  padding: space("large");
}
~~~

Use a function for calculation or validated lookup. Use a mixin when the reusable operation needs to emit a group of declarations or rules.

## 10. Use `@extend` with care

`@extend` changes selectors that contain the extended selector so they also match the extending selector. It is appropriate when there is a real semantic relationship, often through a placeholder:

~~~scss
%message-base {
  padding: 0.75rem 1rem;
  border-radius: 0.375rem;
}

.message {
  @extend %message-base;
}

.message--error {
  @extend %message-base;
  color: #8c1d2c;
}
~~~

Placeholder selectors beginning with `%` do not appear in CSS unless something extends them. Extensions can change selector lists in places other than where the `@extend` was written. This can make generated selectors harder to predict.

For configurable or purely mechanical style reuse, a mixin is often easier to understand. Do not choose `@extend` only because it appears to reduce duplicated source text.

## 11. Control the cascade deliberately

File order and selector scope both affect the cascade. Keep the entry point’s load order understandable, and avoid relying on a long chain of overrides to repair earlier component rules.

If the application uses cascade layers, declare the layer order in one clear location and assign foundational, component, and utility styles deliberately. Treat the layer plan as a project-level CSS decision; Sass modules organize source files but do not replace the browser’s cascade.

Do not add layers to every project by habit. Use a layer strategy when it makes ordering easier to reason about.

## 12. Avoid hidden global state

Avoid depending on the old behavior where variables and mixins become available everywhere through `@import`. With `@use`, a file’s dependencies are explicit and module members have namespaces.

A file should be understandable by reading:

1. its `@use` statements,
2. its public variables, functions, or mixins,
3. the selectors it defines.

This makes it easier to refactor a module without relying on which stylesheet happened to load first.

## 13. Keep output and source boundaries aligned

A source file can define reusable Sass members, emit CSS, or do both. Decide which role it has. Common patterns include:

- Value-only settings module with tokens and lookup functions.
- Component stylesheet that emits one component’s selectors.
- Utility stylesheet that emits a small set of reusable classes.
- Entry point that loads the chosen modules to create the application bundle.

If a file is loaded solely for a variable but also emits unrelated CSS, separate the values from the rules or expose a narrower API.

## 14. Naming and size guidelines

- Use filenames that describe their content, such as `_button.scss` or `_forms.scss`.
- Prefer one component or one coherent responsibility per module.
- Keep public APIs small and stable.
- Avoid generic names such as `_misc.scss` or `_helpers.scss` when their contents can be named more clearly.
- Avoid file-per-selector fragmentation; group rules that change together.
- Keep selector nesting shallow and avoid repeating the full DOM hierarchy.
- Keep private implementation members private when they should not be dependencies.
- Delete obsolete modules and unused mixins when refactoring is complete.

These are maintenance guidelines, not Sass syntax requirements. Adjust the structure to the scale and workflow of the project.

## 15. Migrating from `@import`

Sass’s module system is designed to replace the legacy global `@import` model. Migration is safer when done incrementally:

1. Identify the actual entry points that are compiled.
2. Convert value and helper files to modules first.
3. Add explicit `@use` dependencies to each consumer.
4. Add namespaces and fix references rather than relying on globals.
5. Review generated CSS for duplicate or reordered output.
6. Remove old import paths after every consumer has moved.

Do not simply replace every `@import` string mechanically. A legacy import may depend on global variables, ordering, or nested CSS behavior that needs a deliberate module design.

## Common mistakes

### Loading a partial as though it were an application entry point

A partial can be loaded by another module. Compile the designated app entry point so shared dependencies and CSS order are controlled in one place.

### Expecting module members to be global

`@use` members are scoped to the file that loads them. Add the dependency to every Sass file that needs it and use the namespace.

### Forwarding the whole codebase

A barrel file that forwards every internal member creates a broad, hard-to-change API. Forward only what consumers should rely on.

### Making every component a mixin

If the component is always emitted when loaded, a selector in the component module is direct and easy to find. Reserve mixins for reusable, configurable output.

### Nesting selectors too deeply

Deep nesting hides the final selector and can tie styles to markup structure. Keep component selectors short and intentional.

### Using `@extend` for any repeated declaration

Extension modifies selector relationships; it is not just textual deduplication. Prefer a mixin for configurable or non-semantic style reuse.

### Compiling every partial separately

This can create duplicated dependencies and CSS. Compile the entry points intended for the application bundles.

## Quick reference

| Need | Use |
| --- | --- |
| Load a dependency with a namespace | `@use "path/to/module" as name;` |
| Re-export a module API | `@forward "path/to/module";` |
| Reuse configurable declarations | `@mixin` and `@include` |
| Calculate or return one Sass value | `@function` and `@return` |
| Express a semantic selector relationship | Carefully scoped `@extend` |
| Create non-output extension targets | Placeholder selector such as `%message-base` |
| Control application CSS composition | A single, explicit entry point |
| Share values without implicit globals | Namespaced Sass modules |

## Further reading

- [Sass modules and `@use`](https://sass-lang.com/documentation/at-rules/use/)
- [The `@forward` rule](https://sass-lang.com/documentation/at-rules/forward/)
- [Mixins and `@include`](https://sass-lang.com/documentation/at-rules/mixin/)
- [The `@extend` rule](https://sass-lang.com/documentation/at-rules/extend/)
- [Placeholder selectors](https://sass-lang.com/documentation/style-rules/placeholder-selectors/)
- [Sass module system](https://sass-lang.com/blog/the-module-system-is-launched/)

## Next lesson

Continue with [Project structure and naming conventions](042-project-structure-and-naming-conventions.md).
