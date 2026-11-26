# SCSS quick reference and glossary - Q&A

These questions review the final chapter's syntax reminders and terminology. Examples use modern Dart Sass with SCSS syntax.

## Core concepts

### 1. What is SCSS?

SCSS is one syntax for Sass. It uses braces and semicolons like CSS and accepts ordinary CSS syntax, with Sass features such as variables, modules, mixins, and functions added. The indented Sass syntax is a separate syntax.

### 2. What does Sass do in a front-end project?

Sass is usually a build-time stylesheet language. A Sass compiler reads source files and emits CSS. The browser applies that CSS; it does not evaluate Sass variables, mixins, or functions.

### 3. What is Dart Sass?

Dart Sass is the primary actively maintained implementation of Sass. It is available as a command-line interface and as a package that can be used in a project build.

### 4. What is the difference between a Sass file and a CSS file?

A Sass file may contain Sass-only syntax. A CSS file is the stylesheet output understood by browsers. The build step compiles Sass source into CSS, often with a source map for debugging.

### 5. What is a Sass partial?

A partial is a Sass source file intended to be loaded by another Sass file. Its filename conventionally starts with an underscore, such as <code>_tokens.scss</code>. The underscore is part of the filename; it is usually omitted when referring to the module URL.

## Variables and values

### 6. How are Sass variables written?

They begin with a dollar sign:

~~~scss
$brand: #2457c5;

.button {
  background: $brand;
}
~~~

The compiler substitutes the variable's Sass value while generating CSS.

### 7. How do Sass variables differ from CSS custom properties?

Sass variables are resolved during compilation. CSS custom properties use names beginning with two hyphens, participate in the browser cascade, can inherit, and are read with <code>var()</code>. Use Sass for build-time values and CSS custom properties for values that need runtime theming or local overrides.

### 8. What does the !default flag do?

It assigns a variable only if it has not already been given a non-null value. Libraries use it to provide defaults that consumers can configure:

~~~scss
$brand: royalblue !default;
~~~

For a module, consumers can provide a value through the module configuration syntax when that module is first loaded.

### 9. Should !global be used often?

No. It changes a global variable from inside a nested scope and can make the result depend on hidden mutation. Prefer passing arguments, returning values from functions, or configuring modules explicitly.

### 10. What values can Sass represent?

Common Sass values include numbers, strings, colors, booleans, <code>null</code>, lists, and maps. A number may include units. Sass can calculate compatible units; CSS functions such as <code>calc()</code> are useful when the browser needs to combine values that Sass cannot resolve at build time.

### 11. What are lists and maps?

A list is an ordered collection of values. A map associates keys with values. Lists are useful for sequences; maps are useful for named tokens, such as spacing steps or breakpoints. The <code>sass:list</code> and <code>sass:map</code> modules provide namespaced helpers.

### 12. When should interpolation be used?

Use interpolation, written <code>#{...}</code>, when an expression must become part of CSS text, such as a selector or a property name. For an ordinary property value, write the variable directly when possible:

~~~scss
$color: navy;

.card {
  color: $color;
}
~~~

Avoid interpolation when it adds no value; unnecessary string conversion can make Sass expressions less clear.

## Selectors and nesting

### 13. What does the parent selector & mean?

Inside a nested rule, <code>&</code> stands for the outer selector. It can build a state, element, or modifier selector, as in <code>&:hover</code>, <code>&__title</code>, and <code>&--compact</code>.

### 14. Does Sass nesting change how the browser interprets CSS?

No. Nesting is authoring syntax. Sass combines the selectors and emits CSS; the browser sees the compiled selectors. Deep nesting can create long selectors and increase specificity, so keep nesting shallow and review the generated output.

### 15. Can Sass nest at-rules?

Sass can preserve and nest many CSS at-rules, including media and supports rules. The resulting CSS still follows the semantics of the CSS at-rule. Sass-specific module rules such as <code>@use</code> have their own placement requirements.

## Modules

### 16. What does @use do?

<code>@use</code> loads another Sass stylesheet as a module. It makes the module's public variables, functions, and mixins available under a namespace and includes that module's CSS at most once in a compilation.

~~~scss
@use "tokens";

.button {
  color: tokens.$brand;
}
~~~

### 17. Where should @use appear?

Put <code>@use</code> near the beginning of a file, before style rules and most other statements. Sass permits variables before a configured <code>@use</code>; consult the module documentation when using configuration.

### 18. What does @forward do?

<code>@forward</code> exposes public members from another stylesheet through a module's API. It lets consumers load one entry point instead of knowing the library's internal file layout. If the forwarding stylesheet also needs to use those members internally, it should add its own <code>@use</code>.

### 19. What is a Sass namespace?

A namespace is the prefix used to access members loaded with <code>@use</code>. By default, Sass derives it from the module URL. An alias can be chosen with <code>as</code>. Namespaces make the source of a variable or function visible.

### 20. Is using @use as * a good default?

Usually not for shared or larger stylesheets. It removes the namespace and makes it harder to tell where a name came from or whether two modules define the same name. Explicit namespaces are easier to maintain.

### 21. How do module configuration and !default work together?

The module declares a configurable variable with <code>!default</code>. The consumer supplies an override when loading the module with <code>@use ... with (...)</code>. Module configuration is fixed on its first load in that compilation, so configure it there rather than expecting later loads to change it.

### 22. How are private module members marked?

A variable, mixin, function, or placeholder whose name starts with a hyphen or underscore is private to its Sass module. For example, <code>$-internal-map</code> communicates that consumers should use the public API instead.

### 23. Is Sass @import the same as CSS @import?

No. Sass used <code>@import</code> to load Sass source into a shared global namespace. That Sass feature is deprecated in Dart Sass starting with version 1.80.0. A plain CSS import such as <code>@import url("theme.css");</code> is a CSS feature for loading CSS and is distinct.

## Reusable code and flow control

### 24. When should a mixin be used?

Use a mixin when including it should emit reusable declarations or nested rules. Mixins can accept arguments, and can accept a caller-provided content block through <code>@content</code>.

### 25. When should a Sass function be used?

Use a function when a Sass expression needs a computed value, such as a token lookup or calculation. A function returns its result with <code>@return</code>. Keep functions focused on returning values instead of emitting CSS or modifying global state.

### 26. What is the difference between a mixin and a function?

A mixin is included as a statement and commonly emits styles. A function is called inside an expression and returns a value. Choose by the result you need: declarations or a value.

### 27. How does @extend differ from a mixin?

<code>@extend</code> changes selector relationships: Sass adds the extending selector to rules that match the extended selector. A mixin includes its contents in the current context. Use a mixin when explicit declaration reuse is preferable; use extension only when the selector relationship is intentional.

### 28. What is a placeholder selector?

A placeholder begins with <code>%</code>. Sass does not emit a placeholder-only rule into CSS. When another selector extends it, Sass emits the relevant combined selector rules.

### 29. Which flow-control rules does Sass provide?

Sass provides <code>@if</code>, <code>@each</code>, <code>@for</code>, and <code>@while</code>. They conditionally evaluate code or repeat it. <code>@each</code> is useful for iterating through a list or map; <code>@for</code> repeats over a numeric range.

### 30. What is the difference between @for through and @for to?

<code>through</code> includes the ending number in the range. <code>to</code> stops before the ending number. Check the chosen boundary when generating indexed selectors or spacing classes to avoid an off-by-one result.

## Built-in modules and CSS functions

### 31. How should Sass built-in functions be called?

Load the built-in module and call its function through the module namespace, for example:

~~~scss
@use "sass:map";
@use "sass:math";

$gap: map.get((small: 0.5rem, large: 1.5rem), small);
$half: math.div(1rem, 2);
~~~

This makes the function's origin clear. Sass is deprecating global aliases for built-in functions; use namespaced forms where available.

### 32. Which built-in Sass modules are commonly useful?

- <code>sass:color</code> works with Sass color values.
- <code>sass:list</code> works with lists.
- <code>sass:map</code> works with maps.
- <code>sass:math</code> works with numbers and units.
- <code>sass:meta</code> provides introspection and module-related helpers.
- <code>sass:selector</code> parses and transforms selectors.
- <code>sass:string</code> works with strings.

### 33. Should Sass calculate every CSS value?

No. Use Sass for values known at build time. Preserve CSS functions such as <code>calc()</code>, <code>min()</code>, <code>max()</code>, <code>clamp()</code>, and <code>var()</code> when the browser needs to resolve the value or combine it with runtime information.

## Tooling and diagnostics

### 34. What does the Sass CLI compile command look like?

A typical command names an input Sass file and an output CSS file:

~~~sh
npx sass src/scss/main.scss dist/main.css
~~~

The exact paths depend on the application. Keep the command in a project script or build configuration so it is repeatable.

### 35. What does the --watch option do?

It watches a source file or directory and recompiles when Sass files change. For example, <code>npx sass --watch src/scss:dist</code> watches a directory and writes CSS output to another directory.

### 36. What is the difference between expanded and compressed output?

Expanded output uses readable formatting that is convenient during development. Compressed output removes unnecessary whitespace to reduce file size. Choose output style based on the build and deployment needs.

### 37. What are @error, @warn, and @debug for?

- <code>@error</code> stops compilation with a message, useful when input is invalid.
- <code>@warn</code> reports a problem but lets compilation continue.
- <code>@debug</code> prints a value for development diagnostics.

Remove temporary debug statements before finishing a feature.

### 38. What is a source map?

A source map connects generated CSS locations to their Sass source locations. With source maps available and developer tools configured, a developer can inspect the original source while debugging browser styles.

## Troubleshooting and review

### 39. Why is a Sass variable unavailable in another file?

With the module system, each file must load the module whose members it needs. Add an appropriate <code>@use</code>, then refer to the member through its namespace. A file's module scope is not automatically global.

### 40. Why might a CSS custom property fail inside a media-query condition?

CSS custom properties are substituted in CSS property values; they are not general variables for media-query conditions. Keep query thresholds in Sass or build configuration, and use custom properties for declarations that need runtime values.

### 41. Why did @extend create unexpected selectors?

An extension changes selector rules across the relevant module graph. It is not a simple textual copy. Inspect the compiled selectors, keep extension relationships intentional, and choose a mixin when emitted declarations should be explicit.

### 42. What should be reviewed in compiled CSS?

Check that selectors are no deeper or more specific than intended, that no unwanted Sass helpers were emitted, that CSS custom properties and at-rules remain in place, and that output paths and source maps match the project setup.

### 43. How can a team keep Sass builds reproducible?

Pin the Sass dependency through the project's package manager, record build and watch scripts in project configuration, use a clear entry point and output path, and avoid making manual changes to generated CSS.

### 44. What is a good workflow when a Sass deprecation appears?

Read the warning to identify the deprecated construct, consult the matching official migration guide, update the Sass source or dependency, and run the project's normal build. Treat warnings as actionable maintenance information rather than suppressing them by default.

### 45. What is the fastest way to choose the right Sass feature?

- Compile-time lookup or calculation: variable, map, or function.
- Reusable emitted styles: mixin.
- Public module API: <code>@forward</code>.
- Importing a dependency into the current file: <code>@use</code>.
- Runtime theme or inherited browser value: CSS custom property.
- Conditional or repeated CSS generation: a small flow-control block.
- Intentional selector relationship: carefully considered <code>@extend</code>.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="053-capstone-build-a-reusable-scss-design-system.md">053 - Capstone: build a reusable SCSS design system - Q&amp;A</a></td><td><a href="../lessons/054-scss-quick-reference-and-glossary.md">Lesson</a></td><td><a href="../examples/054-scss-quick-reference-and-glossary.md">Example</a></td><td>End of course</td></tr>
  </tbody>
</table>
