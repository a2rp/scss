# SCSS Keywords and Concepts

A practical glossary for Sass and SCSS syntax. It covers Sass directives, values, operators, modules, functions, and terms often confused with regular CSS.

This is a companion reference to the [SCSS learning roadmap](README.md). Sass evolves; check the official documentation linked below for version-sensitive behavior.

## Contents

- [Sass, SCSS, and syntax](#sass-scss-and-syntax)
- [Stylesheet structure](#stylesheet-structure)
- [Variables and configuration](#variables-and-configuration)
- [SassScript values and operators](#sassscript-values-and-operators)
- [Selectors, nesting, and interpolation](#selectors-nesting-and-interpolation)
- [Sass at-rules](#sass-at-rules)
- [The module system](#the-module-system)
- [Built-in modules and functions](#built-in-modules-and-functions)
- [CSS features used from SCSS](#css-features-used-from-scss)
- [Legacy syntax and deprecations](#legacy-syntax-and-deprecations)
- [Files, compilation, and tooling](#files-compilation-and-tooling)
- [Quick syntax example](#quick-syntax-example)
- [Official references](#official-references)

## Sass, SCSS, and syntax

Sass is a stylesheet language that compiles to CSS. It has two syntaxes: SCSS, which looks like CSS and uses `.scss`, and the indentation-based syntax, which uses `.sass`. This repository uses SCSS for most examples.

Sass does not have one short, closed list of reserved words. Some terms are Sass-specific directives; others are expression keywords, module members, or ordinary CSS syntax. Their meaning depends on where they appear.

| Term | Meaning |
| --- | --- |
| Sass | The stylesheet language and its compiler ecosystem. |
| SCSS | The CSS-like Sass syntax using braces and semicolons. Most valid CSS is valid SCSS. |
| Indented syntax | Sass's indentation-based syntax, usually saved with the `.sass` extension. |
| Dart Sass | The actively maintained Sass implementation. This roadmap follows the version noted in [README](README.md). |
| SassScript | Sass's expression language for values, variables, calculations, and function calls. |
| Sass-specific syntax | Syntax such as `$variables`, modules, mixins, and control flow that is evaluated during compilation. |
| CSS syntax in SCSS | Ordinary CSS rules, properties, values, functions, and at-rules that Sass preserves or processes according to their grammar. |
| CSS output | The stylesheet emitted by Sass and consumed by a browser. |

## Stylesheet structure

| Term | Meaning |
| --- | --- |
| Statement | A unit of Sass source, such as a variable declaration, style rule, or at-rule. |
| Style rule | A selector followed by a declaration block, such as `.card { ... }`. |
| Selector | A pattern that matches elements or other stylesheet targets. |
| Declaration | A property and value inside a style rule, such as `color: navy`. |
| Declaration block | Statements enclosed in `{}` in SCSS. |
| At-rule | An instruction beginning with `@`, such as Sass's `@use` or CSS's `@media`. |
| Sass expression | A value or calculation evaluated by Sass, such as `math.div(12px, 2)`. |
| Interpolation | Inserting a Sass expression into text with `#{...}`. |
| Mixin include | A request to insert a mixin's contents with `@include`. |
| Module member | A variable, function, or mixin exposed by a module. |
| Compilation | Processing Sass source into CSS, including evaluating Sass-only syntax. |
| Source map | A mapping from generated CSS back to source files for developer tools. |

SCSS statements generally end with semicolons. A semicolon is not required after a block. Whitespace and comments separate tokens; whitespace inside quoted strings is part of the string.

## Variables and configuration

A Sass variable starts with `$` and is evaluated at compile time.

~~~scss
$brand-color: #3157c8;
$space: 1rem;

.card {
  color: $brand-color;
  padding: $space;
}
~~~

| Syntax or term | Meaning |
| --- | --- |
| `$name: value` | Declares or assigns a Sass variable. |
| `$name` | Reads a Sass variable. Variables are lexically scoped and can be reassigned. |
| `!default` | Assigns a value only if the variable is not already set or is `null`. Commonly used for configurable library defaults. |
| `!global` | Assigns a variable in the global scope from a nested scope. Use sparingly. |
| Module configuration | Values passed through `@use ... with (...)` to configure public variables marked `!default`. |
| Private member | A module member whose name begins with `-` or `_`; it is not exposed as part of the module's public API. |
| Hyphen/underscore equivalence | Sass treats hyphens and underscores as equivalent in Sass identifiers. Prefer one spelling consistently. |
| CSS custom property | A runtime CSS variable such as `--brand-color`. It is not a Sass variable and is read by the browser. |
| `var(--name)` | A CSS function that reads a custom property at runtime. |

Sass variables disappear into generated CSS after compilation. CSS custom properties remain in CSS and can vary at runtime, inherit through the DOM, and respond to the cascade.

## SassScript values and operators

| Value or term | Meaning |
| --- | --- |
| Number | A number with or without units, such as `12`, `1.5`, `10px`, or `2rem`. |
| Unit | A suffix on a number. Sass calculates with compatible units and detects incompatible ones. |
| String | Text that may be quoted (`"menu"`) or unquoted (`solid`). |
| Color | A CSS color value, such as `rebeccapurple`, `#3157c8`, or `rgb(20 40 60)`. |
| Boolean | `true` or `false`, used in conditions and comparisons. |
| `null` | Sass's absence value; distinct from CSS's `none`. |
| List | An ordered sequence of values separated by spaces, commas, or slashes. A single value can also behave as a one-item list. |
| Map | Key/value pairs, for example `("small": 12px, "large": 20px)`. Maps also behave as lists of pairs. |
| Calculation | A CSS-compatible calculation such as `calc(100% - 2rem)`. |
| Separator | A list delimiter: `space`, `comma`, `slash`, or `auto`. |
| Truthiness | Only `false` and `null` are falsey in Sass. Values such as `0`, `""`, and empty lists are truthy. |

### Operators and expression keywords

| Syntax | Meaning |
| --- | --- |
| `+`, `-`, `*`, `/`, `%` | Arithmetic operators. Use `math.div()` for Sass numeric division; slash also appears in CSS values. |
| `==`, `!=` | Equality and inequality comparisons. |
| `<`, `<=`, `>`, `>=` | Comparison operators. |
| `and`, `or`, `not` | Boolean logic used in conditions. |
| `()` | Groups an expression or represents an empty list. |
| `:` | Separates keys and values in maps and declarations. |
| `...` | Rest arguments in function/mixin definitions or argument forwarding. |
| `$name: value` in a call | Supplies a named argument. |
| `!important` | A CSS declaration flag, not a Sass control-flow keyword. |

Sass preserves slash-separated CSS values where their meaning belongs to CSS, such as `font: 16px/1.5`. Use `math.div($a, $b)` when the intent is mathematical division.

### Control-flow words

| Word | Meaning |
| --- | --- |
| `if` / `else` | Selects conditional branches in `@if` rules. |
| `each` | Iterates over list or map values with `@each`. |
| `for` | Repeats with a numeric counter using `@for`. |
| `from` | Introduces the starting number in a `@for` loop. |
| `through` | Includes the ending number in a `@for` range. |
| `to` | Stops a `@for` range before its ending number. |
| `while` | Repeats an `@while` block while its condition is true. |
| `in` | Separates a loop variable from the values or range being traversed. |

These words are meaningful in Sass control-flow grammar; they are not general-purpose declarations or CSS property names.

## Selectors, nesting, and interpolation

| Syntax or term | Meaning |
| --- | --- |
| `&` | The parent selector inside a nested rule or selector expression. |
| `#{expression}` | Interpolates a Sass value into a selector, property name, at-rule, or string. |
| Nested rule | A selector written inside another rule. Sass combines the selectors in generated CSS. |
| Selector list | A comma-separated set of selectors. Nesting can combine each outer selector with each inner selector. |
| Compound selector | Multiple simple selectors targeting the same element, such as `.card.is-open`. |
| Descendant selector | A space between selectors, matching a nested descendant. |
| Parent selector suffix | Text appended to `&`, often used for BEM-style names such as `&__title`. |
| Placeholder selector | A selector beginning with `%`, emitted only when extended. |
| `@extend` | Shares matching rules with another selector; unlike a mixin, it changes selector relationships instead of copying declarations. |
| `@at-root` | Emits nested styles outside selected enclosing selectors or at-rules. |

~~~scss
.card {
  &__title { color: navy; }
  &:hover { color: royalblue; }
}
~~~

This compiles to selectors such as `.card__title` and `.card:hover`. Nesting is a source-authoring feature; browsers receive ordinary CSS selectors.

## Sass at-rules

A rule beginning with `@` is not necessarily Sass-specific: `@media`, for example, is a CSS at-rule that Sass understands and preserves.

| Sass at-rule | Purpose |
| --- | --- |
| `@use` | Loads a Sass module and exposes its public members through a namespace. |
| `@forward` | Loads a module and re-exports selected public members. |
| `@mixin` | Defines reusable Sass statements. |
| `@include` | Includes a mixin's statements at the current location. |
| `@content` | Inserts a block supplied to a mixin by its caller. |
| `@function` | Defines a Sass function that returns a value. |
| `@return` | Returns a value from a Sass function. |
| `@extend` | Extends a selector's matching rules with another selector. |
| `@at-root` | Emits nested rules at the stylesheet root or outside selected enclosing rules. |
| `@if` / `@else if` / `@else` | Conditionally emits statements based on expressions. |
| `@each` | Loops over list or map values. |
| `@for` | Loops through a numeric range using `from` with `to` or `through`. |
| `@while` | Repeats while a condition remains true. |
| `@error` | Stops compilation and reports an error. |
| `@warn` | Prints a warning without stopping compilation. |
| `@debug` | Prints a value for debugging. |

`@use` and `@forward` are module-loading rules. `@mixin` and `@function` definitions belong at the top level of a stylesheet, not inside ordinary style rules.

## The module system

The module system organizes Sass members and replaces the older global `@import` model.

~~~scss
@use "sass:math";
@use "tokens";

$half-space: math.div(tokens.$space, 2);
~~~

| Term or syntax | Meaning |
| --- | --- |
| Module | A Sass stylesheet loaded with `@use` or `@forward`. |
| Namespace | The prefix used to access a module's members, usually derived from its filename. |
| `@use "tokens"` | Loads a module and exposes members through the `tokens.` namespace. |
| `@use "tokens" as t` | Loads a module with a shorter namespace, so members use `t.`. |
| `@use "tokens" as *` | Loads members without a namespace. This can cause naming conflicts. |
| `@use "tokens" with (...)` | Configures public variables marked `!default` when loading a module. |
| `@forward "tokens"` | Re-exports public members for consumers of the current module. |
| `show` / `hide` | Restricts which members an `@use` or `@forward` exposes. |
| `as prefix-*` | Adds a prefix to members re-exported by `@forward`. |
| Module CSS | CSS emitted by a module. A module is loaded once per compilation, so its CSS is included once. |
| Load path | A configured directory Sass searches when resolving a module URL. |
| Sass URL | The path or URL in `@use` or `@forward`. Sass can resolve partial names and extensions. |
| Configuration variable | A public `!default` variable set by a consumer through `with (...)`. |

Prefer explicit namespaces such as `math.div()` and `map.get()`. They show where a member comes from and help avoid collisions with CSS functions and user-defined functions.

## Built-in modules and functions

Load built-in Sass modules with `@use`. Their URLs start with `sass:`.

| Module | Main use | Example members |
| --- | --- | --- |
| `sass:color` | Color inspection, conversion, and adjustment. | `color.adjust()`, `color.change()`, `color.scale()`, `color.mix()` |
| `sass:list` | Creating, inspecting, and modifying lists. | `list.append()`, `list.join()`, `list.nth()`, `list.length()` |
| `sass:map` | Creating and querying maps. | `map.get()`, `map.set()`, `map.merge()`, `map.has-key()` |
| `sass:math` | Numeric calculations and unit inspection. | `math.div()`, `math.round()`, `math.ceil()`, `math.is-unitless()`, `math.compatible()` |
| `sass:meta` | Inspecting Sass values and module behavior. | `meta.type-of()`, `meta.inspect()`, `meta.module-variables()`, `meta.load-css()` |
| `sass:selector` | Parsing, combining, and inspecting selectors. | `selector.nest()`, `selector.append()`, `selector.unify()` |
| `sass:string` | Quoting, indexing, and transforming strings. | `string.quote()`, `string.unquote()`, `string.index()`, `string.slice()` |

| Function | Module | Typical use |
| --- | --- | --- |
| `math.div($a, $b)` | `sass:math` | Divides Sass numbers while respecting unit rules. |
| `math.round()`, `math.min()`, `math.max()`, `math.clamp()` | `sass:math` | Rounds or constrains numeric values. |
| `math.unit()`, `math.is-unitless()`, `math.compatible()` | `sass:math` | Inspects number units. |
| `color.adjust()` / `color.scale()` | `sass:color` | Changes color channels by fixed amounts or proportionally. |
| `map.get()`, `map.has-key()`, `map.merge()` | `sass:map` | Reads, checks, or combines map entries. |
| `list.nth()`, `list.length()`, `list.append()` | `sass:list` | Reads, counts, or adds list items. |
| `string.quote()` / `string.unquote()` | `sass:string` | Changes whether a string is quoted. |
| `meta.type-of()` / `meta.inspect()` | `sass:meta` | Inspects a value's type or representation. |
| `selector.nest()` / `selector.unify()` | `sass:selector` | Combines selectors using Sass's selector engine. |

These are common examples, not a list of every function. Use the official module reference for complete signatures and compatibility notes.

## CSS features used from SCSS

Sass accepts CSS rules and many CSS at-rules. Sass-specific directives are evaluated by the compiler; CSS syntax generally remains in the emitted stylesheet.

| CSS feature | Meaning in SCSS |
| --- | --- |
| `@media` | CSS media query; Sass supports nesting and interpolation where the grammar permits. |
| `@supports` | CSS feature query. |
| `@container` | CSS container query. |
| `@layer` | CSS cascade layer. |
| `@font-face` | CSS font-face declaration. |
| `@keyframes` | CSS animation steps. |
| `@property` | CSS registration for a typed custom property. |
| `calc()` | Browser calculation; Sass may simplify compatible values and preserves CSS calculations when needed. |
| `min()`, `max()`, `clamp()` | CSS math functions, distinct from similarly named Sass module functions. |
| `color-mix()` | CSS function for mixing colors at runtime. |
| `var()` | Reads a CSS custom property at runtime. |
| `env()` | Reads a user-agent environment value, such as a safe-area inset. |

The browser does not receive Sass variables, mixin definitions, or Sass control-flow rules.

## Legacy syntax and deprecations

| Legacy term | Current guidance |
| --- | --- |
| Sass `@import` | Deprecated in Dart Sass 1.80.0. Prefer `@use` and `@forward` for Sass modules. CSS `@import` remains a CSS feature when used as plain CSS syntax. |
| Global built-in function | Calls such as `map-get(...)` or `unitless(...)` are deprecated when corresponding module functions exist. Prefer `map.get(...)` or `math.is-unitless(...)`. |
| Sass `if()` function | Deprecated in Dart Sass 1.95.0. Prefer an `@if` rule or a CSS conditional function when that is intended. |
| LibSass / Node Sass | Legacy implementations that do not support the current Dart Sass module system. Use Dart Sass for new work. |
| Legacy importer | Older tool integrations may need changes to resolve Sass modules correctly. |

Warnings identify code that may stop working in a future release. Read the warning and official migration guide before changing a legacy codebase; not every CSS `@import` is a Sass module import.

## Files, compilation, and tooling

| Term | Meaning |
| --- | --- |
| Partial | A Sass file conventionally prefixed with `_`, such as `_tokens.scss`, intended to be loaded by another stylesheet rather than compiled as an entry point. |
| Entry point | The top-level Sass file compiled by the build command. |
| Generated CSS | Browser-ready output emitted by the compiler. |
| Expanded output | Readable CSS with declarations and rules on separate lines. |
| Compressed output | Minified CSS intended to reduce file size. |
| Watch mode | Compiler mode that recompiles when source files change. |
| Sass migrator | Official CLI tool that helps migrate legacy Sass syntax, including `@import` to modules. |
| `sass` package | Dart Sass's npm package and command-line compiler. |
| `npx sass` | Runs the locally installed Sass command-line tool from an npm project. |
| Deprecation warning | A compiler warning that a feature is deprecated and may be removed in a future release. |

A partial is a naming convention, not a security boundary. A file can still be compiled directly if passed as an entry point.

## Quick syntax example

~~~scss
@use "sass:math";

$spacing: 1rem;

@mixin stack($gap: $spacing) {
  display: flex;
  gap: $gap;
}

.card {
  padding: math.div($spacing * 3, 2);
  @include stack();

  &__title {
    color: rebeccapurple;
  }

  @media (width >= 48rem) {
    padding: $spacing * 2;
  }
}
~~~

Sass evaluates the variable, module call, mixin, and nesting, then emits ordinary CSS.

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass syntax](https://sass-lang.com/documentation/syntax/)
- [Sass at-rules](https://sass-lang.com/documentation/at-rules/)
- [Built-in modules](https://sass-lang.com/documentation/modules/)
- [Sass `@use`](https://sass-lang.com/documentation/at-rules/use/)
- [Sass `@mixin` and `@include`](https://sass-lang.com/documentation/at-rules/mixin/)
- [Sass `@import` and global built-in deprecations](https://sass-lang.com/documentation/breaking-changes/import/)

## Related files

- [Course roadmap](README.md)
