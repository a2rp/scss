# 033 - Generated CSS and Entry Points: Q&A

## Output fundamentals

### 1. What is a Sass entrypoint?

An entrypoint is a Sass stylesheet that a CLI or build tool compiles directly into a CSS output. It usually loads reusable modules with @use or @forward.

~~~scss
// app.scss
@use "base/reset";
@use "components/card";
~~~

### 2. What does Sass compile when I compile app.scss?

Sass compiles app.scss and the modules it loads into the requested CSS output. Sass variables, namespaces, and mixin definitions are processed during compilation; the browser receives CSS.

### 3. Does @use include a module's CSS?

Yes. @use makes a module's members available and includes the module's CSS in the compilation output.

### 4. Does loading a partial include its CSS?

Yes. If a partial contains top-level CSS and an entrypoint loads it, that CSS is included. The leading underscore is about standalone compilation, not about suppressing CSS when used as a module.

### 5. Do variables by themselves create CSS?

No. A variable assignment provides a Sass value. CSS is emitted only when that value is used in CSS-producing syntax such as a declaration.

### 6. Does defining a mixin emit its declarations?

No. The mixin's declarations are emitted when some stylesheet includes that mixin.

~~~scss
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
}
~~~

Nothing calls this mixin yet, so it contributes no CSS selector on its own.

### 7. Where does included mixin CSS appear?

The declarations appear where the @include is written in the generated CSS, within the selector or at-rule that surrounds the include.

### 8. What does “@use loads a module once” mean?

Within one compilation, Sass evaluates a module once and includes that module's CSS once, even if it is loaded through multiple dependencies in the same module graph.

### 9. Does once-per-module mean identical selectors are deduplicated?

No. Sass does not merge equivalent rules from different source modules. Once-only loading avoids re-emitting one module's CSS when that same module is loaded repeatedly.

### 10. What happens if two different modules define the same selector?

Both rules are emitted. The browser applies the normal CSS cascade, including source order and specificity. If the duplicate is unintended, locate and consolidate the source rules.

### 11. In what order does Sass emit CSS?

CSS is emitted according to how modules are loaded and the order of rules in the source. Since order can affect the cascade, keep entrypoint @use and @forward rules intentional.

### 12. Does Sass scope CSS selectors to a module namespace?

No. The namespace from @use applies to Sass variables, functions, and mixins. CSS selectors remain ordinary selectors and are emitted as written.

### 13. Does forwarding a module emit its CSS?

It can. @forward loads the forwarded module, so top-level CSS in that module can be included in the entrypoint's output.

### 14. Does loading a module from another module include its CSS transitively?

Yes. CSS from modules in the loaded module graph can become part of the entrypoint's output. Inspect each module's top-level rules when output seems unexpected.

### 15. Does a plain CSS file loaded with @use contribute CSS?

Yes. Sass can load a CSS file, and its CSS is included in the output. A CSS file does not provide Sass variables, mixins, or functions.

## Controlling when CSS is emitted

### 16. How can I make a component's CSS opt-in?

Put its declarations in a mixin and include that mixin inside the intended component selector. Defining or loading the mixin does not emit those declarations until it is included.

### 17. Should every module avoid top-level CSS?

No. Top-level CSS is appropriate when loading the module should intentionally add those rules, such as a reset or global typography layer. Keep tokens and helper modules free of unintended global CSS.

### 18. Why did loading a token module add CSS?

The module or one of its dependencies probably contains top-level style rules. Search the module graph for selectors, at-rules, or forwarded modules that emit CSS.

### 19. Can a component partial contain a top-level selector?

Yes, but it will be emitted whenever the module is loaded, whether or not the component appears in a page. Use a mixin if the component CSS should only be included at a deliberate location.

### 20. Can I make two different entrypoints share one Sass module?

Yes. Each entrypoint can @use the same module. Each entrypoint is compiled independently, so the module's CSS can appear once in each separate output.

## Entry points and CLI compilation

### 21. How do I compile one entrypoint?

Use Dart Sass one-to-one mode:

~~~bash
npx sass src/scss/app.scss dist/css/app.css
~~~

### 22. How do I compile several separate entrypoints?

Compile each one to its own output, or use Dart Sass many-to-many mode with explicit input-output pairs.

~~~bash
npx sass src/scss/app.scss dist/css/app.css
npx sass src/scss/admin.scss dist/css/admin.css
~~~

### 23. How do I compile a directory?

Use a source-directory and destination-directory pair:

~~~bash
npx sass src/scss:dist/css
~~~

Dart Sass compiles Sass files in the input directory tree to CSS outputs with corresponding names.

### 24. Are partials compiled as standalone files in directory mode?

No. Dart Sass ignores files whose names begin with an underscore when compiling whole directories. Those files can still be loaded by entrypoints.

### 25. Should I put experiments in the entrypoint directory?

Be careful. Directory compilation treats non-partial Sass files as inputs. Use partial names for modules and keep only intentional entrypoints as non-partial files, or compile the intended entrypoints explicitly.

### 26. Why is the same reset CSS in app.css and admin.css?

Those are separate compilations. Each output includes the reset module once in its own module graph.

### 27. Should I load both generated bundles on one page?

Only if that page needs both. If both include shared CSS, the browser will see the shared rules from both files. Consider which bundles each page actually requires.

## Output formatting and source maps

### 28. What is the default CSS output style?

Dart Sass uses expanded CSS by default, with selectors and declarations formatted for readability.

### 29. How do I create compressed output?

Pass --style=compressed:

~~~bash
npx sass --style=compressed src/scss/app.scss dist/css/app.css
~~~

This changes the output formatting and whitespace. It does not intentionally change the stylesheet's behavior.

### 30. Does Dart Sass create source maps?

Yes. When writing CSS files, Dart Sass generates source maps by default. A source map connects generated CSS to the Sass files that contributed to it.

### 31. How do I disable source maps with the CLI?

Pass --no-source-map:

~~~bash
npx sass --no-source-map src/scss/app.scss dist/css/app.css
~~~

### 32. Does a source map add more CSS rules?

No. It is a separate debugging file or mapping reference. It helps tools locate source Sass but does not add styling rules to the CSS.

## Debugging and design choices

### 33. How do I trace an unexpected CSS rule?

Start at the compiled entrypoint, follow its @use and @forward rules, then inspect each loaded module for top-level rules and included mixins. A source map can help map the generated declaration back to its source.

### 34. How can I prevent global selectors from being emitted automatically?

Avoid top-level selectors in modules that should only provide reusable values or behavior. Put component declarations in a mixin and include it where the component selector is defined.

### 35. Does Sass minify or merge duplicate declarations automatically?

Do not rely on Sass to optimize equivalent rules. Choose the output style you need and use the CSS processing or minification step documented by your project.

### 36. Should I compile every Sass file separately?

Usually not. Compile intentional application or library entrypoints. Partials and internal modules should be loaded by those entrypoints.

### 37. What should I check when CSS seems missing?

Confirm the right entrypoint is being compiled, that the module is loaded, and that the CSS isn't only inside a mixin that has not been included. Then check the build output path and whether the browser loaded that file.

### 38. What should I check when CSS appears twice?

Check whether the same selector is written in different modules, whether separate bundles both include shared styles, or whether the page loaded the same output file more than once. @use only guarantees once-per-module within one compilation.

## Quick reference

~~~scss
// Module CSS is included when loaded.
@use "base";

// A mixin definition is opt-in.
@mixin card {
  padding: 1rem;
}

.card {
  @include card;
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
- [Example: Generated CSS and entry points](../examples/033-emitted-css-and-entrypoints.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="032-partials-load-paths-and-file-naming.md">032 - Partials, Load Paths, and File Naming: Q&amp;A</a></td><td><a href="../lessons/033-emitted-css-and-entrypoints.md">Lesson</a></td><td><a href="../examples/033-emitted-css-and-entrypoints.md">Example</a></td><td><a href="034-legacy-import-and-global-built-in-deprecations.md">034 - Legacy @import and Global Built-In Deprecations: Q&amp;A</a></td></tr>
  </tbody>
</table>
