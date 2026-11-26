# 033 - Generated CSS and Entry Points

## Learning goals

After this lesson, you should be able to:

- Explain how a Sass entrypoint and its loaded modules combine into CSS.
- Identify which Sass constructs emit CSS and which only define reusable members.
- Predict the order of CSS from modules and the entrypoint.
- Explain why @use includes a module's CSS once per compilation.
- Distinguish module loading from including a mixin.
- Design one or more entrypoints for an application or library.
- Compile a single stylesheet or a source directory with Dart Sass.
- Inspect and control the generated CSS output style.

## 1. Sass compiles an entrypoint and its dependency graph

A Sass project often splits source across modules. The entrypoint is the stylesheet you ask Sass to compile directly. It loads other stylesheets with @use or @forward.

~~~text
src/scss/
├── app.scss
├── _reset.scss
├── _tokens.scss
└── components/
    └── _card.scss
~~~

~~~scss
// src/scss/_reset.scss
* {
  box-sizing: border-box;
}

// src/scss/_tokens.scss
$text-color: #263238;
$surface-color: #ffffff;

// src/scss/components/_card.scss
@use "../tokens";

.card {
  color: tokens.$text-color;
  background-color: tokens.$surface-color;
  padding: 1rem;
}

// src/scss/app.scss
@use "reset";
@use "components/card";

body {
  margin: 0;
}
~~~

Compile app.scss:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

Sass compiles the entrypoint together with the modules it loads. The output is CSS; variables, namespaces, partial filenames, and Sass-only mixin calls do not remain in the browser stylesheet.

## 2. CSS rules at the top level are emitted when a module loads

A module can contain ordinary style rules. When a consumer loads that module, its CSS becomes part of the compilation output.

~~~scss
// src/scss/_banner.scss
.site-banner {
  padding: 1rem;
  background-color: #1769aa;
  color: white;
}

// src/scss/app.scss
@use "banner";
~~~

Generated CSS:

~~~css
.site-banner {
  padding: 1rem;
  background-color: #1769aa;
  color: white;
}
~~~

The same applies to CSS loaded indirectly through a module dependency or public entrypoint. A partial filename prevents standalone output in directory compilation, but it does not suppress CSS when the partial is loaded.

## 3. Sass members alone do not create CSS

Variables, functions, and mixin definitions describe reusable Sass values and behavior. Defining a mixin does not emit its declarations until a stylesheet includes it.

~~~scss
// src/scss/_buttons.scss
@mixin primary {
  display: inline-flex;
  padding: 0.6rem 1rem;
  color: white;
  background-color: #1769aa;
}

// src/scss/app.scss
@use "buttons";
~~~

With only the @use, there is no .button selector in the output because the module has no top-level CSS and the mixin was not included.

Include it when the component needs the declarations:

~~~scss
// src/scss/app.scss
@use "buttons";

.button {
  @include buttons.primary;
}
~~~

Generated CSS:

~~~css
.button {
  display: inline-flex;
  padding: 0.6rem 1rem;
  color: white;
  background-color: #1769aa;
}
~~~

This is a useful distinction when designing a library: loading definitions can stay quiet, while mixins provide opt-in CSS at the selector where they are included.

## 4. @use includes each module's CSS once per compilation

Sass's module system evaluates a module once in a compilation, even if multiple modules load it.

~~~text
src/scss/
├── app.scss
├── _base.scss
├── _header.scss
└── _footer.scss
~~~

~~~scss
// src/scss/_base.scss
html {
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
@use "base";
@use "header";
@use "footer";
~~~

The html rule from _base.scss appears once in app.css even though app.scss, _header.scss, and _footer.scss all load the same module.

This is per compilation. A different entrypoint that is compiled separately has its own compilation and can emit the same module CSS into its own output file.

## 5. Once per module does not mean duplicate rules disappear

Sass does not deduplicate identical CSS written in different modules.

~~~scss
// src/scss/_header.scss
.site-title {
  font-weight: 700;
}

// src/scss/_footer.scss
.site-title {
  font-weight: 700;
}

// src/scss/app.scss
@use "header";
@use "footer";
~~~

Both .site-title rules appear in the generated CSS because they are two distinct rules from two different source modules. @use's once-only behavior prevents one module from being evaluated repeatedly; it does not optimize equivalent selectors across different files.

If repeated output is unintended, find the source rules and centralize or remove them. Do not rely on Sass to merge selectors or declarations automatically.

## 6. Output order follows module loading and source order

The order of CSS can affect the cascade. Keep the entrypoint's module order intentional.

~~~scss
// src/scss/_base.scss
.notice {
  color: #263238;
}

// src/scss/_theme.scss
.notice {
  color: #1769aa;
}

// src/scss/app.scss
@use "base";
@use "theme";
~~~

The base rule is emitted before the theme rule. With equal specificity, the later color declaration wins in the cascade.

~~~css
.notice {
  color: #263238;
}

.notice {
  color: #1769aa;
}
~~~

A module's own dependencies are loaded as part of that module's graph. Think about those dependencies when deciding which top-level CSS a public entrypoint will include.

Prefer modules with no top-level CSS for tokens and utilities. Keep intentional global resets and base styles in explicit modules that the application entrypoint includes on purpose.

## 7. CSS from @forward is included too

A public Sass API assembled with @forward can also assemble CSS. Forwarding a module loads it, so its top-level CSS is part of the output.

~~~scss
// src/scss/design-system/_normalize.scss
* {
  box-sizing: border-box;
}

// src/scss/design-system/_buttons.scss
@mixin button {
  padding: 0.6rem 1rem;
}

// src/scss/design-system/_index.scss
@forward "normalize";
@forward "buttons";

// src/scss/app.scss
@use "design-system" as ui;

.action {
  @include ui.button;
}
~~~

Generated CSS includes the universal reset and the .action selector. The mixin contributes declarations at the include site; the normalize module contributes its top-level rule when the entrypoint loads the design-system module.

Keep this side effect in mind when deciding whether a module belongs in a public entrypoint. A consumer may load the API for a variable or mixin and receive its top-level CSS as well.

## 8. CSS output from a module is not scoped to the consumer

A module's top-level selectors are emitted as written. Loading the module inside another module does not automatically nest or scope those selectors.

~~~scss
// src/scss/components/_dialog.scss
.dialog {
  position: fixed;
  inset: 0;
}

// src/scss/app.scss
@use "components/dialog";
~~~

This produces a global .dialog selector. Sass modules scope their Sass members with namespaces, but CSS selectors remain ordinary CSS selectors.

If the CSS should appear only when a component is used, put the declarations in a mixin and include that mixin at the intended selector:

~~~scss
// src/scss/components/_dialog.scss
@mixin styles {
  position: fixed;
  inset: 0;
}

// src/scss/app.scss
@use "components/dialog";

.dialog {
  @include dialog.styles;
}
~~~

Generated CSS:

~~~css
.dialog {
  position: fixed;
  inset: 0;
}
~~~

This separates module availability from the decision to emit a particular component's CSS.

## 9. Use a dedicated application entrypoint

A clear entrypoint makes global CSS and component dependencies visible in one place.

~~~text
src/scss/
├── app.scss
├── base/
│   ├── _reset.scss
│   └── _typography.scss
├── components/
│   ├── _button.scss
│   └── _card.scss
└── tokens/
    ├── _colors.scss
    └── _spacing.scss
~~~

~~~scss
// src/scss/app.scss
@use "base/reset";
@use "base/typography";
@use "components/button";
@use "components/card";
~~~

Component modules should load the tokens they use themselves:

~~~scss
// src/scss/components/_card.scss
@use "../tokens/colors";
@use "../tokens/spacing";

.card {
  padding: spacing.$space-medium;
  color: colors.$text;
  background-color: colors.$surface;
}
~~~

Compile the entrypoint rather than compiling every partial as a standalone stylesheet:

~~~bash
npx sass src/scss/app.scss dist/css/app.css
~~~

This creates one application CSS file from the entrypoint and the modules it loads.

## 10. Multiple entrypoints create separate CSS bundles

An application may intentionally have more than one stylesheet, such as app.scss and admin.scss.

~~~text
src/scss/
├── app.scss
├── admin.scss
├── _tokens.scss
├── _base.scss
└── components/
    ├── _site.scss
    └── _admin-panel.scss
~~~

~~~scss
// src/scss/_base.scss
html {
  box-sizing: border-box;
}

// src/scss/app.scss
@use "base";
@use "components/site";

// src/scss/admin.scss
@use "base";
@use "components/admin-panel";
~~~

Compile each entrypoint into its own output:

~~~bash
npx sass src/scss/app.scss dist/css/app.css
npx sass src/scss/admin.scss dist/css/admin.css
~~~

Each compilation can include the base CSS in its output. That is expected because these files are separate deliverables. On a page, include only the bundle or bundles that page needs.

If both bundles are included on the same page, their shared CSS may appear twice in the browser's combined stylesheets. Decide whether separate bundles are necessary for the pages that consume them.

## 11. Compile several entrypoints in directory mode

Dart Sass can compile all non-partial Sass files in a source directory to a destination directory.

~~~text
src/scss/
├── app.scss
├── admin.scss
├── _tokens.scss
└── components/
    ├── _card.scss
    └── _button.scss
~~~

~~~bash
npx sass src/scss:dist/css
~~~

The output directory receives app.css and admin.css. Underscore-prefixed partials are ignored as standalone inputs, but they are still loaded by entrypoints that use them.

Directory mode is convenient when every non-partial Sass file in the source tree is an intentional entrypoint. If the tree contains experiments or files that should not produce CSS, compile explicit entrypoints or use a build tool's configured input list.

## 12. Choose expanded or compressed output

Dart Sass uses expanded CSS by default, which is readable during development.

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

Expanded output:

~~~css
.card {
  padding: 1rem;
  color: #263238;
}
~~~

Use the compressed style when a deployment pipeline specifically needs compact CSS:

~~~bash
npx sass --style=compressed src/scss/app.scss dist/app.min.css
~~~

Compressed output:

~~~css
.card{padding:1rem;color:#263238}
~~~

Compression changes whitespace and formatting, not the intended stylesheet behavior. Many production build tools minify CSS in a later build step, so use the mechanism documented by the project.

## 13. Source maps and generated files

Dart Sass emits source maps by default for CSS files it writes. A source map helps browser developer tools relate generated CSS back to source Sass files.

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

This can produce:

~~~text
dist/
├── app.css
└── app.css.map
~~~

If your project or deployment process does not need a source map, Sass CLI supports disabling it:

~~~bash
npx sass --no-source-map src/scss/app.scss dist/app.css
~~~

Use the application's build configuration to decide whether maps should be retained, published, or disabled. A source map is a debugging aid; it is not another stylesheet entrypoint.

## 14. Inspect where generated CSS comes from

When output is surprising, trace it from the entrypoint:

1. Identify the exact entrypoint passed to the Sass command or build tool.
2. Read its @use and @forward rules in order.
3. Follow modules' own dependencies.
4. Find top-level style rules in those modules.
5. Find mixins included by the entrypoint or its modules.
6. Check whether another distinct module defines a similar selector.
7. Check whether the CSS came from another output file also loaded by the page.

A source map can help locate the source of a generated declaration in browser developer tools. It does not change which rules Sass emits.

## 15. Common mistakes

### Compiling a partial directly as if it were the application

A partial is designed to be loaded by an entrypoint. Compile the actual non-partial entrypoint or the intended directory.

### Expecting @use to emit no CSS

@use makes Sass members available and includes the loaded module's CSS. Keep top-level rules only in modules whose CSS should be emitted when loaded.

### Expecting a mixin definition to emit declarations

A mixin's declarations are emitted when the mixin is included. Define a mixin without including it and no CSS is generated from that mixin body.

### Assuming Sass deduplicates selectors

@use loads one module once. It does not remove equivalent rules authored in different modules.

### Loading several bundles without checking overlap

Each entrypoint compiles separately. Shared module CSS can be present in each bundle, and the browser applies every bundle loaded by the page.

### Putting global selectors in a component module unintentionally

Top-level selectors remain global when emitted. Use a mixin for CSS that should be emitted only at a chosen component selector.

## 16. Summary

- Compile a deliberate entrypoint; Sass combines its CSS with the modules it loads.
- Top-level CSS in a module is emitted when the module is loaded.
- Defining a variable, function, or mixin alone does not emit selector CSS.
- Including a mixin emits its declarations at the include site.
- @use includes each module's CSS once per compilation, but does not deduplicate rules across different modules.
- CSS ordering follows module loading and source order, so entrypoint order can affect the cascade.
- @forward can bring top-level CSS into a public entrypoint.
- Multiple entrypoints produce separate outputs and separate compilations.
- Directory mode compiles non-partial files as outputs and ignores underscore-prefixed partials as standalone inputs.
- Dart Sass CLI supports expanded or compressed CSS output and emits source maps by default.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="032-partials-load-paths-and-file-naming.md">032 - Partials, Load Paths, and File Naming</a></td><td><a href="../qnas/033-emitted-css-and-entrypoints.md">Q&amp;A</a></td><td><a href="../examples/033-emitted-css-and-entrypoints.md">Example</a></td><td><a href="034-legacy-import-and-global-built-in-deprecations.md">034 - Legacy @import and Global Built-In Deprecations</a></td></tr>
  </tbody>
</table>
