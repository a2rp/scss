# SCSS quick reference and glossary

This chapter is a compact companion to the course. Use it to recall syntax, choose the right Sass feature, and review core terms. It assumes modern Dart Sass and the SCSS syntax.

Sass runs during the build and emits CSS. Sass variables and functions do not continue running in the browser. Use CSS custom properties when a value must participate in the browser cascade or change at runtime.

## 1. SCSS at a glance

SCSS is the CSS-like syntax for Sass. Most valid CSS is also valid SCSS, with Sass features added. Indented Sass is a separate syntax; do not mix its indentation-only rules with SCSS braces and semicolons.

~~~scss
$brand: #2457c5;

.button {
  color: white;
  background: $brand;

  &:hover {
    background: #173f99;
  }

  @media (min-width: 48rem) {
    padding-inline: 1.5rem;
  }
}
~~~

A Sass compiler turns this source into ordinary CSS. The browser applies the generated CSS, not Sass variables or mixins.

## 2. Variables, values, and interpolation

### Variables and flags

~~~scss
$space-unit: 0.25rem;
$card-radius: 0.75rem !default;

.card {
  padding: $space-unit * 4;
  border-radius: $card-radius;
}
~~~

- A variable begins with <code>$</code>.
- Sass variables are imperative: a use gets the value defined at that point in the stylesheet.
- <code>!default</code> assigns a value only when the variable is undefined or <code>null</code>. It is useful for module configuration.
- <code>!global</code> changes a global variable from inside a nested scope. Prefer explicit function results, mixin arguments, or module configuration over hidden global mutation.
- Names beginning with <code>-</code> or <code>_</code> are private to a Sass module. For example, <code>$-internal-scale</code> is not part of the module's public API.
- Hyphens and underscores are equivalent in Sass identifiers. Pick one style and use it consistently.

Sass values include numbers with units, strings, colors, booleans, <code>null</code>, lists, and maps. Sass calculates compatible units, while incompatible units need deliberate conversion or should remain in CSS math such as <code>calc()</code>.

~~~scss
@use "sass:map";

$spacing: (
  small: 0.5rem,
  medium: 1rem,
  large: 1.5rem
);

.panel {
  padding: map.get($spacing, medium);
  width: calc(100% - 2rem);
}
~~~

### Sass variables versus CSS custom properties

| Need | Use |
| --- | --- |
| A value used while generating a stylesheet | Sass variable |
| A value selected by the browser cascade or inherited by descendants | CSS custom property |
| A responsive query threshold known during compilation | Sass variable or build configuration |
| A theme value that changes on a subtree without recompiling | CSS custom property |

~~~scss
$wide-layout: 56rem;

:root {
  --surface: white;
  --text: #172033;
}

[data-theme="dark"] {
  --surface: #172033;
  --text: #f5f7fb;
}

.card {
  color: var(--text);
  background: var(--surface);
}

@media (min-width: $wide-layout) {
  .card {
    padding: 2rem;
  }
}
~~~

A CSS custom property is substituted by the browser in property values. It is not a general replacement for Sass variables in calculations or ordinary media-query conditions.

### Interpolation

Use <code>#{...}</code> when a Sass expression must become part of CSS syntax, such as a selector, property name, or custom-property value. Ordinary property values can usually use a variable directly.

~~~scss
$state: "open";

.menu--#{$state} {
  --menu-state: #{$state};
}
~~~

## 3. Selectors and nesting

Nesting groups related rules. Sass combines nested selectors when it emits CSS.

~~~scss
.card {
  border: 1px solid #d7dce5;

  &__title {
    margin: 0;
  }

  &:focus-within {
    outline: 3px solid #2457c5;
  }

  .card__content {
    padding: 1rem;
  }
}
~~~

The parent selector <code>&</code> represents the outer selector. Common forms include <code>&:hover</code>, <code>&__element</code>, and <code>&--modifier</code>. It can also appear in a nested selector such as <code>[dir="rtl"] &amp;</code>.

Keep nesting shallow and tied to real component relationships. Deep nesting can produce long, high-specificity selectors and make markup changes expensive. Nesting is a source-writing convenience; it does not create a runtime relationship in CSS.

## 4. Modules: @use and @forward

Use modules to make dependencies and public APIs explicit.

~~~scss
// _tokens.scss
$brand: #2457c5 !default;

// app.scss
@use "tokens";

.button {
  background: tokens.$brand;
}
~~~

- <code>@use</code> loads a Sass module and exposes public members under a namespace. A module's CSS is included at most once in a compilation.
- <code>@use</code> rules belong near the top of a stylesheet, before style rules. Variables may be declared before a configured <code>@use</code>.
- The default namespace is based on the module URL. Use <code>as short-name</code> when a clearer or non-conflicting name is useful.
- <code>@use "tokens" as *</code> removes the namespace. It can be convenient in a small isolated file, but makes names harder to trace.
- A module can expose configurable variables with <code>!default</code> and receive values with <code>@use "tokens" with ($brand: navy)</code>. Configure a module on its first load in that compilation.
- A leading <code>-</code> or <code>_</code> marks module members private.

Use <code>@forward</code> to create a deliberate entry point for consumers:

~~~scss
// _index.scss
@forward "tokens";
@forward "buttons";

// consumer.scss
@use "design-system";

.page {
  color: design-system.$brand;
}
~~~

<code>@forward</code> re-exports public variables, functions, and mixins. A module that uses a member internally must also <code>@use</code> it. Put <code>@forward</code> before <code>@use</code> when consumers should be able to configure the forwarded module.

Sass <code>@import</code> is deprecated in Dart Sass 1.80.0. Prefer <code>@use</code> and <code>@forward</code> for Sass modules. A plain CSS import such as <code>@import url("theme.css");</code> remains a CSS feature and is not the same as loading a Sass module.

## 5. Mixins, functions, and extension

### Mixins emit reusable styles

~~~scss
@mixin focus-ring($color: #2457c5) {
  &:focus-visible {
    outline: 3px solid $color;
    outline-offset: 2px;
  }
}

.button {
  @include focus-ring();
}
~~~

A mixin can accept required, optional, and keyword arguments. It can also accept a content block with <code>@content</code>, useful when a caller needs to provide nested styles. Mixins fit when inclusion emits declarations or rules.

### Functions return values

~~~scss
@use "sass:map";

$spaces: (
  1: 0.25rem,
  2: 0.5rem,
  3: 0.75rem
);

@function space($step) {
  $value: map.get($spaces, $step);

  @if $value == null {
    @error "Unknown spacing step: #{$step}.";
  }

  @return $value;
}

.stack {
  gap: space(3);
}
~~~

Use functions to calculate or look up a value. Functions should return a value rather than emit styles or mutate global state.

### Extension changes selector relationships

~~~scss
%control {
  font: inherit;
  border-radius: 0.4rem;
}

.text-input {
  @extend %control;
  padding: 0.6rem 0.8rem;
}
~~~

A placeholder selector starts with <code>%</code> and is omitted from output unless extended. <code>@extend</code> adds selectors to rules that match the extended selector; it does not copy a block like a mixin. Prefer mixins when emitted declarations should be explicit and predictable. Use <code>@extend</code> only when the selector relationship is intentional and the resulting selector grouping is understood.

## 6. Collections and flow control

Lists contain ordered values. Maps associate keys with values. Prefer namespaced functions in <code>sass:list</code> and <code>sass:map</code> for clear, modern code.

~~~scss
@use "sass:map";

$breakpoints: (
  tablet: 48rem,
  desktop: 72rem
);

@each $name, $width in $breakpoints {
  @media (min-width: $width) {
    .show-from-#{$name} {
      display: block;
    }
  }
}
~~~

Sass flow-control rules:

- <code>@if</code> and <code>@else</code> choose whether to evaluate a block.
- <code>@each</code> iterates over values in a list or key-value pairs in a map.
- <code>@for</code> repeats a block over a numeric range. <code>through</code> includes the ending number; <code>to</code> stops before it.
- <code>@while</code> repeats while a condition is true. Use it carefully so the condition eventually becomes false.

Flow control is most useful for small, data-driven CSS generation. If a loop produces a large or hard-to-understand stylesheet, write the needed rules explicitly or move generation to a more appropriate build tool.

## 7. Built-in Sass modules

Load built-in modules with their <code>sass:</code> URL and call members through the namespace.

~~~scss
@use "sass:color";
@use "sass:list";
@use "sass:map";
@use "sass:math";
@use "sass:meta";
@use "sass:selector";
@use "sass:string";
~~~

| Module | Typical use |
| --- | --- |
| <code>sass:color</code> | Inspect and adjust colors |
| <code>sass:list</code> | Read, append, or search list values |
| <code>sass:map</code> | Read and update key-value maps |
| <code>sass:math</code> | Numeric operations and unit helpers |
| <code>sass:meta</code> | Introspection, dynamic member access, and module tools |
| <code>sass:selector</code> | Parse and manipulate selector values |
| <code>sass:string</code> | Search and construct strings |

Use explicit module namespaces, for example <code>map.get($tokens, "brand")</code> or <code>math.div(1rem, 2)</code>. Built-in global function aliases are deprecated; use module forms where available. CSS functions such as <code>calc()</code>, <code>min()</code>, <code>max()</code>, <code>clamp()</code>, and <code>var()</code> should remain CSS functions when the browser needs to resolve them.

## 8. CSS at-rules and diagnostics

Sass supports CSS at-rules and adds Sass-specific ones.

- <code>@media</code>, <code>@supports</code>, <code>@container</code>, <code>@layer</code>, and <code>@keyframes</code> are CSS at-rules that Sass can preserve and, in many cases, nest.
- <code>@use</code> and <code>@forward</code> organize Sass modules.
- <code>@mixin</code>, <code>@include</code>, and <code>@content</code> define and use reusable style blocks.
- <code>@function</code> and <code>@return</code> define value-producing helpers.
- <code>@extend</code> relates selectors; <code>@at-root</code> emits rules outside the current nesting context.
- <code>@error</code> stops compilation with a useful message.
- <code>@warn</code> reports a warning but allows compilation to continue.
- <code>@debug</code> prints a value while developing.

Use errors for invalid input that must prevent a correct build. Use warnings for a recoverable issue that consumers should fix. Remove temporary debug output before considering a feature complete.

## 9. Dart Sass command-line reminders

Install Dart Sass using the method appropriate for the project. In an npm project, record the package and scripts in <code>package.json</code> so teammates can use the same toolchain.

~~~sh
npm install --save-dev sass
npx sass src/scss/main.scss dist/main.css
npx sass --watch src/scss:dist
npx sass --style=expanded src/scss/main.scss dist/main.css
npx sass --style=compressed src/scss/main.scss dist/main.min.css
~~~

- The first path is the input Sass file; the second is the generated CSS file.
- <code>--watch</code> recompiles when watched source files change.
- <code>--style=expanded</code> is easier to inspect. <code>--style=compressed</code> minimizes output whitespace.
- Source maps help browser developer tools map generated CSS back to Sass source. Check the CLI options and bundler configuration for the project's source-map setup.
- Keep generated CSS in the location expected by the application. Do not edit generated output when Sass source is the source of truth.

A project script can make the command repeatable:

~~~json
{
  "scripts": {
    "css:build": "sass src/scss/main.scss public/css/main.css",
    "css:watch": "sass --watch src/scss:public/css"
  }
}
~~~

Then run <code>npm run css:build</code> or <code>npm run css:watch</code>.

## 10. Common mistakes to catch

1. **Treating Sass variables as browser variables.** A Sass variable is replaced during compilation; use a CSS custom property for runtime theming.
2. **Using Sass <code>@import</code> for new modules.** Move to <code>@use</code> and <code>@forward</code>; distinguish these from plain CSS imports.
3. **Using a module member without its namespace.** Add the appropriate <code>@use</code> or re-export it deliberately with <code>@forward</code>.
4. **Putting <code>@use</code> after style rules.** Keep module loads at the beginning of the file.
5. **Assuming nesting changes browser behavior.** Nesting compiles to selectors; review emitted selectors and specificity.
6. **Overusing interpolation.** Use direct variable references in property values; interpolate only where Sass syntax must become CSS text.
7. **Using <code>@extend</code> as if it copied declarations.** It changes selector groups; use a mixin for explicit reusable declarations.
8. **Using global built-in aliases.** Load the relevant <code>sass:</code> module and call its namespaced function.
9. **Generating CSS with complex loops unnecessarily.** Keep generated output small, predictable, and readable.
10. **Editing generated CSS by hand.** Update the source and rebuild so changes are reproducible.

## 11. Glossary

| Term | Meaning |
| --- | --- |
| **Dart Sass** | The primary actively maintained Sass implementation, available as a command-line tool and package. |
| **SCSS** | The brace-and-semicolon Sass syntax that is CSS-like and accepts ordinary CSS syntax. |
| **Sass** | A stylesheet language that extends CSS with variables, modules, functions, mixins, and other authoring features. The word also names the indented Sass syntax and the implementations. |
| **SassScript** | The expression language used for Sass values, calculations, conditions, and function calls. |
| **Compilation** | The build-time process that reads Sass source and emits CSS. |
| **Module** | A Sass stylesheet loaded through the module system, typically with <code>@use</code> or <code>@forward</code>. |
| **Partial** | A Sass source file intended to be loaded by another stylesheet. A leading underscore conventionally marks it, such as <code>_tokens.scss</code>. |
| **Entry point** | The Sass file compiled to a CSS output file; it may load other modules. |
| **Namespace** | The prefix used to access members exposed by <code>@use</code>, such as <code>tokens.$brand</code>. |
| **Public member** | A variable, function, or mixin intended for consumers of a module. |
| **Private member** | A module member whose name begins with <code>-</code> or <code>_</code> and is not exposed as part of the module API. |
| **Configuration** | Values supplied to a module with <code>@use ... with (...)</code>, usually overriding variables marked <code>!default</code>. |
| **Interpolation** | Inserting a Sass expression into text with <code>#{...}</code>. |
| **Nesting** | Writing a selector or at-rule inside another rule so Sass can combine or group the output. |
| **Parent selector** | The <code>&amp;</code> selector that refers to the outer selector in a nested rule. |
| **Mixin** | A reusable Sass construct included with <code>@include</code>; it can emit declarations or nested rules. |
| **Content block** | Caller-provided Sass content passed to a mixin and inserted with <code>@content</code>. |
| **Function** | A Sass construct called in an expression that returns a value with <code>@return</code>. |
| **Placeholder selector** | A selector beginning with <code>%</code> that is omitted from CSS unless another selector extends it. |
| **Extension** | The selector transformation performed by <code>@extend</code>. |
| **List** | An ordered Sass collection. Values can be separated by spaces or commas. |
| **Map** | A Sass collection of key-value pairs, often used for design tokens or configuration. |
| **Built-in module** | A module provided by Sass, such as <code>sass:map</code> or <code>sass:color</code>. |
| **CSS custom property** | A browser-level property declared with <code>--name</code> and read with <code>var(--name)</code>. |
| **Source map** | A mapping file that helps developer tools connect generated CSS back to source files. |
| **Output style** | The formatting mode used when Sass emits CSS, commonly expanded or compressed. |
| **Deprecation warning** | A compiler message that a feature is discouraged and may be removed in a future version. |

## 12. Fast decision guide

- Need a compile-time token or loop input? Use a Sass variable or map.
- Need a value that changes with a theme or inherited browser state? Use a CSS custom property.
- Need to emit a reusable group of declarations? Use a mixin.
- Need to calculate one value? Use a function.
- Need to expose a module's supported API? Use <code>@forward</code>.
- Need to consume another Sass module? Use <code>@use</code>.
- Need conditional or repeated source generation? Use <code>@if</code>, <code>@each</code>, or <code>@for</code> when it keeps output understandable.
- Need to relate selectors intentionally? Consider <code>@extend</code>; otherwise prefer explicit classes or mixins.
- Need to check what the browser receives? Inspect compiled CSS and use source maps in developer tools.

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [SCSS and Sass syntax](https://sass-lang.com/documentation/syntax/)
- [Sass module loading with @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass module forwarding with @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Built-in Sass modules](https://sass-lang.com/documentation/modules/)
- [Mixins and @include](https://sass-lang.com/documentation/at-rules/mixin/)
- [Sass functions](https://sass-lang.com/documentation/at-rules/function/)
- [Flow control](https://sass-lang.com/documentation/at-rules/control/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/)
- [Sass import deprecation](https://sass-lang.com/documentation/breaking-changes/import/)
