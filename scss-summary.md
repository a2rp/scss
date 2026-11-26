# SCSS Summary

A course-wide overview of Sass and SCSS, from compiling a first stylesheet to building maintainable modules, design systems, and production CSS. Use this for review; use the [SCSS keywords and concepts](scss-keywords.md) file when you need a definition of a specific term.

Sass is a stylesheet language that compiles to CSS. SCSS is its CSS-like syntax. The browser does not run Sass: it receives the generated CSS. Sass adds compile-time features such as variables, nesting, mixins, functions, modules, and control flow while continuing to support ordinary CSS.

## Contents

- [1. What Sass and SCSS do](#1-what-sass-and-scss-do)
- [2. Install and compile Sass](#2-install-and-compile-sass)
- [3. SCSS syntax and stylesheet structure](#3-scss-syntax-and-stylesheet-structure)
- [4. Values, variables, and scope](#4-values-variables-and-scope)
- [5. Calculations, lists, and maps](#5-calculations-lists-and-maps)
- [6. Nesting, selectors, and interpolation](#6-nesting-selectors-and-interpolation)
- [7. Mixins, functions, and reusable logic](#7-mixins-functions-and-reusable-logic)
- [8. Modules and file organization](#8-modules-and-file-organization)
- [9. Control flow and generated CSS](#9-control-flow-and-generated-css)
- [10. Sass and CSS work together](#10-sass-and-css-work-together)
- [11. Design systems and framework integration](#11-design-systems-and-framework-integration)
- [12. Debugging, quality, and production delivery](#12-debugging-quality-and-production-delivery)
- [13. Legacy Sass and migration](#13-legacy-sass-and-migration)
- [14. Practical workflow](#14-practical-workflow)
- [15. Roadmap coverage](#15-roadmap-coverage)
- [Official references](#official-references)

## 1. What Sass and SCSS do

Sass is a language for authoring stylesheets. A compiler evaluates Sass-only syntax and emits CSS for browsers. SCSS is the commonly used syntax: it uses braces and semicolons, and most valid CSS is valid SCSS. The older indented Sass syntax uses indentation instead.

Sass is useful when a project benefits from reusable values, shared style patterns, calculations, modules, or generated CSS. It is not a replacement for understanding CSS. The cascade, inheritance, layout, accessibility, responsive behavior, and browser support still determine how the final page behaves.

A typical pipeline is:

1. Write source in one or more SCSS files.
2. Compile an entry stylesheet with Dart Sass or a build tool.
3. Inspect and deliver the generated CSS.
4. Let the browser apply that CSS to the document.

Sass runs at build time. CSS custom properties and CSS functions can still run in the browser at runtime.

## 2. Install and compile Sass

For a project that uses npm, install Sass as a development dependency:

~~~bash
npm install --save-dev sass
~~~

Compile an entry file:

~~~bash
npx sass src/scss/main.scss dist/css/main.css
~~~

Watch a source directory during development:

~~~bash
npx sass --watch src/scss:dist/css
~~~

Dart Sass is the actively maintained implementation. This repository's roadmap states the version it targets; check the official documentation for version-sensitive features and deprecations.

The Sass command can also select output style, source maps, load paths, and warning behavior. Use a framework or build tool's documented Sass integration when the application already has a build pipeline. Keep generated output in the location expected by that pipeline.

## 3. SCSS syntax and stylesheet structure

SCSS uses CSS-like style rules and Sass at-rules:

~~~scss
.card {
  color: navy;
  padding: 1rem;
}
~~~

A stylesheet is made of statements. A style rule has a selector and a declaration block. A declaration sets a CSS property to a value. Sass variables and at-rules can appear in the positions allowed by the Sass grammar.

- End declarations with semicolons.
- Use braces to group rules and declarations.
- Use comments to explain intent; SCSS supports CSS block comments and Sass-only line comments.
- Keep one clear responsibility per stylesheet or module.
- Remember that CSS syntax remains CSS unless Sass specifically evaluates it.

Sass has two syntaxes. SCSS files usually end in .scss. Indented syntax files usually end in .sass and use indentation instead of braces and semicolons. A project can choose either syntax, but mixing both styles in one file is not allowed.

## 4. Values, variables, and scope

A Sass variable starts with a dollar sign and stores a compile-time value:

~~~scss
$brand-color: #3157c8;
$space: 1rem;

.button {
  color: $brand-color;
  padding: $space;
}
~~~

Sass variables can store numbers, strings, colors, booleans, null, lists, and maps. They are evaluated by Sass and usually disappear from the generated CSS.

- Use descriptive names that communicate purpose.
- Keep variables near the module or component that owns them.
- Avoid mutable global variables as a general state mechanism.
- Use !default for configurable library defaults.
- Use !global only when a nested scope truly needs to assign a global variable.
- Sass treats hyphens and underscores as equivalent in Sass identifiers; choose one spelling consistently.

Sass variables differ from CSS custom properties. Sass variables are resolved at compile time. CSS custom properties remain in the browser output, participate in inheritance and the cascade, and can change at runtime.

~~~scss
$brand-color: #3157c8;

:root {
  --brand-color: #{$brand-color};
}

.button {
  color: var(--brand-color);
}
~~~

Interpolation is used here to insert the Sass value into a custom property declaration. Use a CSS custom property when the value must vary by theme, element, user setting, or runtime state.

## 5. Calculations, lists, and maps

SassScript supports arithmetic and functions. Unit-aware calculations help catch mistakes, but CSS and Sass do not evaluate every expression in the same way.

~~~scss
@use "sass:math";

$half: math.div(24px, 2);

.panel {
  width: calc(100% - 2rem);
  gap: $half;
}
~~~

Use namespaced built-in functions from modules such as sass:math, sass:color, sass:list, and sass:map. Division with a slash is also valid CSS syntax, so use math.div() when the intent is Sass numeric division. Leave expressions such as font-size/line-height as CSS when that is their CSS meaning.

Lists hold ordered values and can use spaces, commas, or slashes as separators. Maps associate keys with values and are useful for structured tokens such as breakpoints, colors, or spacing scales. Prefer named map keys when they make the data clearer.

~~~scss
@use "sass:map";

$colors: (
  "brand": #3157c8,
  "text": #18202a
);

.button {
  background: map.get($colors, "brand");
}
~~~

Do not build a large framework of Sass data structures without a concrete need. Keep tokens understandable and make the emitted CSS easy to inspect.

## 6. Nesting, selectors, and interpolation

Nesting lets related selectors be written near one another. The parent selector ampersand represents the current outer selector:

~~~scss
.card {
  &__title {
    color: navy;
  }

  &:hover {
    border-color: royalblue;
  }

  .icon {
    margin-inline-end: 0.5rem;
  }
}
~~~

This emits selectors such as .card__title, .card:hover, and .card .icon. Keep nesting shallow. Deep nesting couples styles tightly to markup and can create long, overly specific selectors.

Interpolation inserts a Sass value into a selector, property name, at-rule, or string. Use it when generating names or values is useful; avoid generating selectors that are difficult to predict or debug.

The parent selector is convenient for modifier and BEM-style names. Use @extend only when selectors should share a relationship in the output. A mixin is usually clearer when the intent is to copy reusable declarations or accept parameters. Excessive extension can make generated selectors difficult to understand.

## 7. Mixins, functions, and reusable logic

A mixin packages declarations or rules that can be included in more than one place. Parameters allow callers to customize it:

~~~scss
@mixin focus-ring($color: royalblue) {
  &:focus-visible {
    outline: 3px solid $color;
    outline-offset: 2px;
  }
}

.button {
  @include focus-ring();
}
~~~

A mixin may accept positional, named, default, and rest arguments. A mixin can also accept a content block through @content, which is useful for controlled wrappers around caller-provided rules. Keep mixins focused and avoid hiding large amounts of unrelated output inside one include.

A Sass function calculates and returns a value. Use functions for value transformations, not as a substitute for CSS behavior that should happen in the browser.

~~~scss
@use "sass:math";

@function rem($pixels, $root-size: 16px) {
  @return math.div($pixels, $root-size) * 1rem;
}
~~~

A function returns a value with @return. A mixin emits reusable stylesheet statements with @include. Choose the abstraction that matches the task.

Use built-in modules instead of global Sass function aliases. Namespaced calls make the function source visible and avoid collisions with CSS functions.

## 8. Modules and file organization

Use the module system to organize Sass files. @use loads a module's public members under a namespace. @forward re-exports public members through a library entry point.

~~~scss
@use "sass:math";
@use "tokens";

.card {
  padding: math.div(tokens.$space, 2);
}
~~~

Useful module rules:

- Put @use and @forward at the beginning of a stylesheet, before ordinary rules. Variable declarations may appear before @use when they configure a loaded module.
- Prefer namespaces such as tokens.$space and math.div().
- Configure a module's public !default variables with @use ... with (...).
- Use @forward to define a module's public API; show, hide, and prefix options can shape that API.
- A module is loaded once per compilation, so its CSS is included once.
- A file prefixed with an underscore is conventionally a partial intended to be loaded by another stylesheet.
- Keep entry points explicit and avoid compiling internal partials as standalone outputs.
- Use load paths deliberately so imports are predictable.

A simple structure might look like:

~~~text
src/
  scss/
    main.scss
    tokens/
      _index.scss
      _colors.scss
      _spacing.scss
    components/
      _button.scss
      _card.scss
~~~

The exact layout should fit the application. Avoid creating many tiny files unless the boundaries improve reuse, ownership, or navigation.

## 9. Control flow and generated CSS

Sass control flow can generate repeated or conditional CSS:

- @if and @else choose statements based on a condition.
- @each iterates through a list or map.
- @for iterates over a numeric range using from with to or through.
- @while repeats while a condition is true.
- @error stops compilation with a useful message.
- @warn reports a concern without stopping compilation.
- @debug prints a value during development.

Only false and null are falsey in Sass. Values such as zero and empty strings are truthy. Make conditions explicit so future readers can see what is being tested.

Generated styles should remain predictable. Use loops for meaningful repeated patterns such as a small utility scale, not to produce huge sets of selectors without a clear need. Inspect the compiled CSS to confirm that loops, nesting, and extensions emit what you expect.

## 10. Sass and CSS work together

SCSS can contain standard CSS features such as media queries, supports queries, container queries, cascade layers, keyframes, custom properties, and CSS functions. Sass processes its own syntax and emits CSS for the browser.

~~~scss
$wide-layout: 48rem;

.page {
  padding: 1rem;

  @media (width >= $wide-layout) {
    padding: 2rem;
  }
}
~~~

Choose between Sass and CSS features based on when a value must be known:

- Use Sass variables for compile-time constants, configuration, and generated stylesheet logic.
- Use CSS custom properties for runtime theming, inheritance, and values that depend on DOM context.
- Use CSS calc(), min(), max(), and clamp() for responsive browser calculations.
- Use Sass math functions for build-time number manipulation.
- Use CSS media and container queries for responsive conditions evaluated by the browser.

Sass and CSS can have functions with similar names. A namespaced Sass function such as math.min() is evaluated by Sass; a CSS function such as min() may be preserved for the browser. Check the output when an expression could be interpreted either way.

## 11. Design systems and framework integration

A design system can use Sass modules for compile-time tokens, mixins, and component styles. CSS custom properties can expose selected tokens at runtime.

A maintainable approach is to:

1. Define a small set of semantic tokens for color, spacing, typography, and breakpoints.
2. Group related tokens and helpers into modules.
3. Expose only the members consumers need through a public module.
4. Keep component styles near the components they describe.
5. Use CSS custom properties for theme values that must change at runtime.
6. Check contrast, focus visibility, reduced motion, and responsive behavior in the browser.
7. Review generated CSS for duplication and excessive selector complexity.

Angular, React, Vite, and other tools can compile Sass through their build systems. Sass itself does not provide component lifecycle behavior, routing, application state, or a UI framework. Follow the framework's guidance for styles, global stylesheets, component styles, and build configuration.

## 12. Debugging, quality, and production delivery

When output is wrong, inspect the source, compiler output, and browser developer tools:

- Read the full Sass error and use its file and line location.
- Use @debug for values during development; remove noisy debugging before delivery.
- Use source maps to connect generated declarations back to SCSS source.
- Check selectors, cascade order, specificity, media conditions, and overridden declarations in developer tools.
- Lint stylesheets and format consistently.
- Review warnings and deprecations rather than silencing them without understanding.
- Test layouts at relevant viewport sizes and with keyboard focus, zoom, and user motion preferences.
- Compile the intended entry points and verify that the deployed application loads the generated CSS.

For production, use the build tool's production configuration. Compressed CSS can reduce output size, but readable source maps and error reporting can still be valuable for development and debugging. Do not edit generated CSS by hand if it is expected to be rebuilt from SCSS.

## 13. Legacy Sass and migration

Sass @import and global aliases for built-in functions are deprecated in Dart Sass. Prefer @use and @forward for Sass modules and namespaced functions such as map.get() and math.div().

For a legacy codebase:

1. Identify whether an @import is a Sass import or a plain CSS import.
2. Review Sass deprecation warnings and identify the owning files.
3. Migrate modules in dependency order, preserving intended CSS output.
4. Use the official Sass migrator when it fits the project.
5. Review changes manually, compile, and compare the generated CSS.
6. Update build-tool integrations if they rely on older importers or implementations.

LibSass and Node Sass do not support the modern Dart Sass module system. New projects should use Dart Sass. A migration can change namespacing, load paths, evaluation, or emitted CSS, so inspect results rather than treating a successful command as proof of identical behavior.

## 14. Practical workflow

1. Start with valid CSS knowledge and a clear styling goal.
2. Create or identify the project's Sass entry point.
3. Install or configure Dart Sass through the project's build tool.
4. Write a small SCSS rule and compile it.
5. Add variables or a module when a repeated value or dependency justifies it.
6. Use mixins for reusable emitted styles and functions for value calculations.
7. Keep nesting shallow and selectors understandable.
8. Use CSS custom properties when browser runtime behavior is needed.
9. Inspect generated CSS and verify behavior in a browser.
10. Address warnings, accessibility, responsiveness, and build integration before delivery.

## 15. Roadmap coverage

The course roadmap contains 54 chapters grouped into these learning areas:

- Foundations: Sass's role, installation, compilation, SCSS syntax, and variables.
- SassScript and values: types, numbers, calculations, strings, lists, maps, colors, and comments.
- Selectors and reusable logic: nesting, interpolation, mixins, functions, built-in modules, control flow, and extension.
- Modules and migration: @use, namespaces, configuration, @forward, partials, generated CSS, deprecations, and migration.
- CSS integration and design systems: CSS at-rules, custom properties, themes, tokens, responsive helpers, component architecture, and naming.
- Tooling and ecosystem: npm scripts, CLI options, framework integrations, and Sass libraries.
- Quality and delivery: diagnostics, linting, source maps, performance, accessibility, and browser support.
- Capstone and reference: a reusable SCSS design system and a quick reference/glossary.

Use the [README roadmap](README.md) to open a chapter's lesson, Q&A, or example. The roadmap identifies planned file paths; a link alone does not mean its file is already complete.
