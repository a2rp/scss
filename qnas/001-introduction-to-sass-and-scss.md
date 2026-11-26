# 001. Q&A: Introduction to Sass and SCSS

Use these questions to review the first lesson. Try to answer each one before reading the explanation.

## Sass and its role

### 1. What is Sass?

Sass is a stylesheet language that adds authoring and organization features to CSS. A Sass compiler processes Sass source and emits CSS that browsers can use.

### 2. What does a Sass compiler produce?

It produces CSS. Sass-only syntax such as variables and nested rules is evaluated or transformed during compilation; the output is ordinary CSS.

### 3. Does a browser execute Sass source?

No. Browsers apply CSS. A project must compile Sass to CSS and load that output, or use a build tool that compiles Sass as part of its build.

### 4. What is the difference between Sass and SCSS?

Sass is the language. SCSS is one of its syntaxes, using CSS-like braces and semicolons. The other commonly used syntax is indentation-based Sass.

### 5. Which file extension is usually used for SCSS?

The `.scss` extension. The indentation-based syntax conventionally uses `.sass`.

### 6. Is ordinary CSS valid SCSS?

Most valid CSS is valid SCSS, because SCSS is designed to be CSS-like. Sass adds additional syntax, so a few CSS edge cases have special handling.

### 7. Does SCSS replace CSS?

No. SCSS is an authoring syntax that compiles into CSS. The browser still applies CSS rules, the cascade, inheritance, layout, and selectors.

### 8. What kinds of features does Sass add?

Sass includes variables, nesting, mixins, functions, modules, and control-flow rules. These help write, reuse, calculate, and organize styles.

### 9. Is Sass a UI framework or component library?

No. Sass focuses on stylesheet authoring and compilation. It does not provide UI components, application state, routing, or page structure.

### 10. Does every website need Sass?

No. Plain CSS may be enough for a small or straightforward project. Use Sass when its features solve a real need and the project can maintain the compiler workflow.

## Compilation and project workflow

### 11. What is the basic Sass workflow?

Write Sass source, compile it to CSS, then load that CSS in the page or application. A build tool can automate the compilation step.

### 12. Why can't I link an SCSS file directly from HTML?

Browsers do not parse or compile Sass. Link the generated CSS file instead, such as `<link rel="stylesheet" href="dist/styles.css">`.

### 13. What does this command do?

~~~bash
npx sass src/styles.scss dist/styles.css
~~~

It runs the Sass CLI to compile `src/styles.scss` and write the resulting CSS to `dist/styles.css`.

### 14. What does watch mode do?

A command such as `npx sass --watch src/scss:dist/css` watches source files and recompiles when they change. It is useful during development.

### 15. Where should the generated CSS go?

Use the output directory expected by the project or its build tool. The HTML or application must refer to the matching output file.

### 16. Should I edit generated CSS by hand?

Usually not when it is produced from Sass. Changes in generated output may be overwritten the next time the compiler runs. Edit the SCSS source and compile again.

### 17. Does every project need a separate manual Sass command?

No. Many build tools or frameworks run Sass automatically. The important point is that compilation occurs before the browser applies the resulting CSS.

## Variables and nesting

### 18. What is a Sass variable?

A named compile-time value whose name begins with `$`, such as `$brand-color: #2457a7;`. Sass substitutes its value where used when it compiles the stylesheet.

### 19. Do Sass variable names appear in generated CSS?

Usually not. Sass resolves the variable and emits its value. For example, `color: $brand-color` may compile to `color: #2457a7`.

### 20. Is a Sass variable the same as a CSS custom property?

No. Sass variables are evaluated during compilation. CSS custom properties, such as `--brand-color`, remain in CSS and are evaluated by the browser at runtime.

### 21. What does nesting do?

Nesting lets related selectors be written inside an outer rule. Sass combines them into regular CSS selectors in the output.

### 22. What does the ampersand mean in a nested selector?

`&` stands for the parent selector. Inside `.card { &:hover { ... } }`, it makes `.card:hover`.

### 23. What happens if I nest a selector without using the ampersand?

Sass usually treats it as a descendant selector. For example, nesting `.card__title` inside `.card` produces `.card .card__title`, with a space between them.

### 24. Is nested SCSS exactly how selectors are sent to the browser?

No. The browser receives the compiled selectors. Nesting is a source-authoring feature, not a runtime browser feature.

### 25. Why should nesting remain shallow?

Deep nesting can create long selectors that depend on markup structure, become harder to reuse, and may have difficult-to-manage specificity. Inspect the emitted CSS and keep relationships intentional.

## Choosing and understanding Sass

### 26. CSS has variables and nesting now. Why use Sass?

Native CSS features cover many needs, and they may be sufficient. Sass can still help with modules, compile-time calculations, mixins, functions, control flow, and established build workflows. Choose based on the project rather than using Sass automatically.

### 27. Does Sass make styles automatically reusable or maintainable?

No. Sass offers tools for reuse and organization, but the author still has to choose clear names, useful boundaries, and predictable output. Excessive nesting or abstraction can make styles harder to maintain.

### 28. What is Dart Sass?

Dart Sass is the primary, actively maintained Sass implementation. Older implementations may lack newer features. For new projects, use Dart Sass and follow the build tool's integration guidance.

### 29. Does Sass change the browser's CSS cascade?

No. Sass emits CSS, and the browser applies the cascade to that CSS. Sass can generate selectors and declarations, but it does not replace browser styling rules.

### 30. How can I tell whether a Sass feature emitted the CSS I expected?

Compile the source, inspect the generated CSS, and load it in a browser. Compare the output selectors and values with the HTML and the intended visual result.

## Scenario checks

### 31. A page shows no styles after you wrote a valid SCSS file. What should you check first?

Check that the SCSS was compiled, that the generated CSS file exists at the expected location, and that the page links to that CSS path.

### 32. You changed a Sass variable but the page still shows the old value. What could be wrong?

The source may not have been recompiled, the page may load a different CSS file, or another declaration may override the generated value. Check the compiler output and browser developer tools.

### 33. Your nested hover rule does not match the intended element. What should you inspect?

Check whether the selector uses `&:hover` for the same element or a nested selector that creates a descendant relationship. Inspect the compiled CSS to see the exact selector.

### 34. A project has only a few simple rules and no repeated values. Is Sass required?

No. Plain CSS may be simpler. Add Sass when its features improve the project's authoring or organization enough to justify compilation.

## Quick review

| Statement | Correct? | Why |
| --- | --- | --- |
| Browsers compile SCSS files directly. | No | Browsers apply CSS; Sass must compile to CSS first. |
| SCSS is a syntax of Sass. | Yes | Sass is the language, and SCSS is its CSS-like syntax. |
| Sass variables are automatically runtime CSS variables. | No | Sass values are resolved during compilation; CSS custom properties remain in the browser. |
| Nesting changes the source and emits CSS selectors. | Yes | Nesting is transformed by Sass into ordinary CSS. |
| Every project needs Sass. | No | Use Sass when its features are useful for that project. |
| The browser's cascade still applies to compiled Sass output. | Yes | The output is CSS, so normal browser CSS rules apply. |

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass syntax: SCSS and indented syntax](https://sass-lang.com/documentation/syntax/)
- [Sass basics](https://sass-lang.com/guide/)
- [Install Sass](https://sass-lang.com/install/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/)
