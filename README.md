# Sass and SCSS: Practical Notes

These are my personal learning notes for Sass, with a focus on the SCSS syntax. The roadmap moves from the compiler and language basics to reusable modules, maintainable design systems, framework integration, and practical projects.

Sass is a stylesheet language that compiles to CSS. SCSS is its CSS-like syntax: most valid CSS is also valid SCSS. The indented `.sass` syntax is covered for comparison, while examples primarily use `.scss` files.

## Learning outcomes

By following this roadmap, you will learn to:

- Set up Dart Sass and compile SCSS into CSS locally and through a project build.
- Use variables, Sass values, calculations, interpolation, nesting, mixins, functions, and control flow appropriately.
- Build namespaced stylesheet modules with `@use` and `@forward`.
- Organize tokens, themes, component styles, and reusable libraries.
- Integrate Sass with common frontend build tools and framework projects.
- Migrate legacy Sass while understanding deprecated features and migration output.
- Debug, lint, and maintain the generated CSS, including accessibility and performance concerns.

## Learning path

The table is the complete planned chapter sequence. Lesson, Q&A, and example files will be added chapter by chapter.

| No. | Lesson | Q&A | Example |
| ---: | --- | --- | --- || 01 | [Introduction to Sass and SCSS](lessons/01-introduction-to-sass-and-scss.md) | [Open Q&A](qnas/01-introduction-to-sass-and-scss.md) | [Open Example](examples/01-introduction-to-sass-and-scss.md) |
| 02 | [How Sass fits CSS and a web build](lessons/02-how-sass-fits-css-and-a-web-build.md) | [Open Q&A](qnas/02-how-sass-fits-css-and-a-web-build.md) | [Open Example](examples/02-how-sass-fits-css-and-a-web-build.md) |
| 03 | [Installing Dart Sass and version basics](lessons/03-installing-dart-sass-and-version-basics.md) | [Open Q&A](qnas/03-installing-dart-sass-and-version-basics.md) | [Open Example](examples/03-installing-dart-sass-and-version-basics.md) |
| 04 | [Compiling SCSS to CSS](lessons/04-compiling-scss-to-css.md) | [Open Q&A](qnas/04-compiling-scss-to-css.md) | [Open Example](examples/04-compiling-scss-to-css.md) |
| 05 | [SCSS syntax and rules](lessons/05-scss-syntax-and-rules.md) | [Open Q&A](qnas/05-scss-syntax-and-rules.md) | [Open Example](examples/05-scss-syntax-and-rules.md) |
| 06 | [Sass variables and scope](lessons/06-sass-variables-and-scope.md) | [Open Q&A](qnas/06-sass-variables-and-scope.md) | [Open Example](examples/06-sass-variables-and-scope.md) |
| 07 | [Sass value types and null](lessons/07-sass-value-types-and-null.md) | [Open Q&A](qnas/07-sass-value-types-and-null.md) | [Open Example](examples/07-sass-value-types-and-null.md) |
| 08 | [Numbers, units, and arithmetic](lessons/08-numbers-units-and-arithmetic.md) | [Open Q&A](qnas/08-numbers-units-and-arithmetic.md) | [Open Example](examples/08-numbers-units-and-arithmetic.md) |
| 09 | [Slash division and calculations](lessons/09-slash-division-and-calculations.md) | [Open Q&A](qnas/09-slash-division-and-calculations.md) | [Open Example](examples/09-slash-division-and-calculations.md) |
| 10 | [Strings and interpolation](lessons/10-strings-and-interpolation.md) | [Open Q&A](qnas/10-strings-and-interpolation.md) | [Open Example](examples/10-strings-and-interpolation.md) |
| 11 | [Lists and list functions](lessons/11-lists-and-list-functions.md) | [Open Q&A](qnas/11-lists-and-list-functions.md) | [Open Example](examples/11-lists-and-list-functions.md) |
| 12 | [Maps and map functions](lessons/12-maps-and-map-functions.md) | [Open Q&A](qnas/12-maps-and-map-functions.md) | [Open Example](examples/12-maps-and-map-functions.md) |
| 13 | [Colors and the Sass color module](lessons/13-colors-and-sass-color-module.md) | [Open Q&A](qnas/13-colors-and-sass-color-module.md) | [Open Example](examples/13-colors-and-sass-color-module.md) |
| 14 | [Comments and generated CSS output](lessons/14-comments-and-css-output.md) | [Open Q&A](qnas/14-comments-and-css-output.md) | [Open Example](examples/14-comments-and-css-output.md) |
| 15 | [Nesting rules and declarations](lessons/15-nesting-rules-and-declarations.md) | [Open Q&A](qnas/15-nesting-rules-and-declarations.md) | [Open Example](examples/15-nesting-rules-and-declarations.md) |
| 16 | [The parent selector](lessons/16-parent-selector.md) | [Open Q&A](qnas/16-parent-selector.md) | [Open Example](examples/16-parent-selector.md) |
| 17 | [Interpolation in selectors and property names](lessons/17-interpolation-in-selectors-and-property-names.md) | [Open Q&A](qnas/17-interpolation-in-selectors-and-property-names.md) | [Open Example](examples/17-interpolation-in-selectors-and-property-names.md) |
| 18 | [Mixin basics](lessons/18-mixin-basics.md) | [Open Q&A](qnas/18-mixin-basics.md) | [Open Example](examples/18-mixin-basics.md) |
| 19 | [Mixin arguments, defaults, and keyword arguments](lessons/19-mixin-arguments-defaults-and-keyword-args.md) | [Open Q&A](qnas/19-mixin-arguments-defaults-and-keyword-args.md) | [Open Example](examples/19-mixin-arguments-defaults-and-keyword-args.md) |
| 20 | [Content blocks and context-aware mixins](lessons/20-content-blocks-and-context-aware-mixins.md) | [Open Q&A](qnas/20-content-blocks-and-context-aware-mixins.md) | [Open Example](examples/20-content-blocks-and-context-aware-mixins.md) |
| 21 | [Custom functions and return values](lessons/21-custom-functions-and-return-values.md) | [Open Q&A](qnas/21-custom-functions-and-return-values.md) | [Open Example](examples/21-custom-functions-and-return-values.md) |
| 22 | [Built-in Sass modules](lessons/22-built-in-sass-modules.md) | [Open Q&A](qnas/22-built-in-sass-modules.md) | [Open Example](examples/22-built-in-sass-modules.md) |
| 23 | [Conditionals and Boolean logic](lessons/23-conditionals-and-boolean-logic.md) | [Open Q&A](qnas/23-conditionals-and-boolean-logic.md) | [Open Example](examples/23-conditionals-and-boolean-logic.md) |
| 24 | [Loops: each, for, and while](lessons/24-loops-each-for-and-while.md) | [Open Q&A](qnas/24-loops-each-for-and-while.md) | [Open Example](examples/24-loops-each-for-and-while.md) |
| 25 | [Generating selectors and utility classes](lessons/25-generating-selectors-and-utilities.md) | [Open Q&A](qnas/25-generating-selectors-and-utilities.md) | [Open Example](examples/25-generating-selectors-and-utilities.md) |
| 26 | [@extend and placeholder selectors](lessons/26-extend-placeholder-selectors.md) | [Open Q&A](qnas/26-extend-placeholder-selectors.md) | [Open Example](examples/26-extend-placeholder-selectors.md) |
| 27 | [@at-root and nested at-rules](lessons/27-at-root-and-at-rules.md) | [Open Q&A](qnas/27-at-root-and-at-rules.md) | [Open Example](examples/27-at-root-and-at-rules.md) |
| 28 | [The module system with @use](lessons/28-module-system-with-use.md) | [Open Q&A](qnas/28-module-system-with-use.md) | [Open Example](examples/28-module-system-with-use.md) |
| 29 | [Module namespaces and private members](lessons/29-module-namespaces-and-private-members.md) | [Open Q&A](qnas/29-module-namespaces-and-private-members.md) | [Open Example](examples/29-module-namespaces-and-private-members.md) |
| 30 | [Module configuration and !default](lessons/30-module-configuration-and-defaults.md) | [Open Q&A](qnas/30-module-configuration-and-defaults.md) | [Open Example](examples/30-module-configuration-and-defaults.md) |
| 31 | [Forwarding public APIs with @forward](lessons/31-forwarding-public-apis-with-forward.md) | [Open Q&A](qnas/31-forwarding-public-apis-with-forward.md) | [Open Example](examples/31-forwarding-public-apis-with-forward.md) |
| 32 | [Partials, load paths, and file naming](lessons/32-partials-load-paths-and-file-naming.md) | [Open Q&A](qnas/32-partials-load-paths-and-file-naming.md) | [Open Example](examples/32-partials-load-paths-and-file-naming.md) |
| 33 | [Generated CSS and entry points](lessons/33-emitted-css-and-entrypoints.md) | [Open Q&A](qnas/33-emitted-css-and-entrypoints.md) | [Open Example](examples/33-emitted-css-and-entrypoints.md) |
| 34 | [Legacy @import and global built-in deprecations](lessons/34-legacy-import-and-global-built-in-deprecations.md) | [Open Q&A](qnas/34-legacy-import-and-global-built-in-deprecations.md) | [Open Example](examples/34-legacy-import-and-global-built-in-deprecations.md) |
| 35 | [Migrating to Sass modules](lessons/35-migrating-to-sass-modules.md) | [Open Q&A](qnas/35-migrating-to-sass-modules.md) | [Open Example](examples/35-migrating-to-sass-modules.md) |
| 36 | [Sass with CSS at-rules](lessons/36-sass-and-css-at-rules.md) | [Open Q&A](qnas/36-sass-and-css-at-rules.md) | [Open Example](examples/36-sass-and-css-at-rules.md) |
| 37 | [Sass variables versus CSS custom properties](lessons/37-sass-variables-vs-css-custom-properties.md) | [Open Q&A](qnas/37-sass-variables-vs-css-custom-properties.md) | [Open Example](examples/37-sass-variables-vs-css-custom-properties.md) |
| 38 | [Runtime theming and design tokens](lessons/38-runtime-theming-and-design-tokens.md) | [Open Q&A](qnas/38-runtime-theming-and-design-tokens.md) | [Open Example](examples/38-runtime-theming-and-design-tokens.md) |
| 39 | [Reusable responsive mixins](lessons/39-reusable-responsive-mixins.md) | [Open Q&A](qnas/39-reusable-responsive-mixins.md) | [Open Example](examples/39-reusable-responsive-mixins.md) |
| 40 | [Maps for design systems](lessons/40-maps-for-design-systems.md) | [Open Q&A](qnas/40-maps-for-design-systems.md) | [Open Example](examples/40-maps-for-design-systems.md) |
| 41 | [Component styles and architecture](lessons/41-component-styles-and-architecture.md) | [Open Q&A](qnas/41-component-styles-and-architecture.md) | [Open Example](examples/41-component-styles-and-architecture.md) |
| 42 | [Project structure and naming conventions](lessons/42-project-structure-and-naming-conventions.md) | [Open Q&A](qnas/42-project-structure-and-naming-conventions.md) | [Open Example](examples/42-project-structure-and-naming-conventions.md) |
| 43 | [npm scripts and build integration](lessons/43-npm-scripts-and-build-integration.md) | [Open Q&A](qnas/43-npm-scripts-and-build-integration.md) | [Open Example](examples/43-npm-scripts-and-build-integration.md) |
| 44 | [Sass CLI, watch mode, and build options](lessons/44-sass-cli-watch-and-build-options.md) | [Open Q&A](qnas/44-sass-cli-watch-and-build-options.md) | [Open Example](examples/44-sass-cli-watch-and-build-options.md) |
| 45 | [Framework integrations: Angular, React, and Vite](lessons/45-framework-integrations-angular-react-vite.md) | [Open Q&A](qnas/45-framework-integrations-angular-react-vite.md) | [Open Example](examples/45-framework-integrations-angular-react-vite.md) |
| 46 | [Authoring a Sass library](lessons/46-authoring-a-sass-library.md) | [Open Q&A](qnas/46-authoring-a-sass-library.md) | [Open Example](examples/46-authoring-a-sass-library.md) |
| 47 | [Warnings, errors, and debugging](lessons/47-warnings-errors-and-debugging.md) | [Open Q&A](qnas/47-warnings-errors-and-debugging.md) | [Open Example](examples/47-warnings-errors-and-debugging.md) |
| 48 | [Linting and stylesheet quality](lessons/48-linting-and-style-quality.md) | [Open Q&A](qnas/48-linting-and-style-quality.md) | [Open Example](examples/48-linting-and-style-quality.md) |
| 49 | [Source maps and browser developer tools](lessons/49-source-maps-and-browser-devtools.md) | [Open Q&A](qnas/49-source-maps-and-browser-devtools.md) | [Open Example](examples/49-source-maps-and-browser-devtools.md) |
| 50 | [CSS output, performance, and maintenance](lessons/50-css-output-performance-and-maintenance.md) | [Open Q&A](qnas/50-css-output-performance-and-maintenance.md) | [Open Example](examples/50-css-output-performance-and-maintenance.md) |
| 51 | [Accessible motion and responsive patterns](lessons/51-accessible-motion-and-responsive-patterns.md) | [Open Q&A](qnas/51-accessible-motion-and-responsive-patterns.md) | [Open Example](examples/51-accessible-motion-and-responsive-patterns.md) |
| 52 | [Browser support and progressive enhancement](lessons/52-browser-support-and-progressive-enhancement.md) | [Open Q&A](qnas/52-browser-support-and-progressive-enhancement.md) | [Open Example](examples/52-browser-support-and-progressive-enhancement.md) |
| 53 | [Capstone: build a reusable SCSS design system](lessons/53-capstone-build-a-reusable-scss-design-system.md) | [Open Q&A](qnas/53-capstone-build-a-reusable-scss-design-system.md) | [Open Example](examples/53-capstone-build-a-reusable-scss-design-system.md) |
| 54 | [SCSS quick reference and glossary](lessons/54-scss-quick-reference-and-glossary.md) | [Open Q&A](qnas/54-scss-quick-reference-and-glossary.md) | [Open Example](examples/54-scss-quick-reference-and-glossary.md) |
## Setup in a practice project

Install the current Dart Sass package in a project that uses Node.js:

```bash
npm install --save-dev sass
```

Compile a stylesheet and watch for changes with the Sass CLI:

```bash
npx sass src/scss/main.scss dist/css/main.css
npx sass --watch src/scss:dist/css
```

In an Angular, React, Vite, or other application, prefer the framework or build tool's documented Sass integration. This repository is a set of learning notes, not a runnable Sass project.

## Modern Sass guidance

- Prefer Dart Sass, the actively maintained Sass implementation.
- Use `@use` and `@forward` for new module APIs instead of relying on global imports.
- Sass `@import` and global built-in function calls are deprecated in Dart Sass starting with version 1.80.0; the roadmap includes migration guidance.
- Sass compiles styles; browser behavior still depends on the CSS output and the browser's support for each CSS feature.
- Keep compiled output understandable, avoid unnecessarily deep selector nesting, and test accessibility and responsive behavior in the resulting interface.

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass syntax](https://sass-lang.com/documentation/syntax/)
- [Sass modules: `@use`](https://sass-lang.com/documentation/at-rules/use/)
- [Sass modules: `@forward`](https://sass-lang.com/documentation/at-rules/forward/)
- [Dart Sass CLI](https://sass-lang.com/documentation/cli/)
- [Sass `@import` deprecation](https://sass-lang.com/documentation/breaking-changes/import/)
