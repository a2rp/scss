# Sass and SCSS: Practical Notes

These are my personal learning notes for Sass, with a focus on the SCSS syntax. This practical roadmap moves from the compiler and language basics to reusable modules, maintainable design systems, framework integration, and projects.

Sass is a stylesheet language that compiles to CSS. SCSS is its CSS-like syntax: most valid CSS is also valid SCSS. The indented `.sass` syntax is covered for comparison, while examples primarily use `.scss` files.

## Quick guides

- [SCSS summary](scss-summary.md)
- [SCSS keywords and concepts](scss-keywords.md)

## Learning outcomes

By following the roadmap, you will learn to:

- Set up Dart Sass and compile SCSS into CSS locally and through a project build.
- Use variables, Sass values, calculations, interpolation, nesting, mixins, functions, and control flow appropriately.
- Build namespaced stylesheet modules with `@use` and `@forward`.
- Organize tokens, themes, component styles, and reusable libraries.
- Integrate Sass with common frontend build tools and framework projects.
- Migrate legacy Sass while understanding deprecated features and migration output.
- Debug, lint, and maintain generated CSS, including accessibility and performance concerns.

## Before you begin

Basic CSS and HTML knowledge will help you understand how Sass output is used in a page. Familiarity with a terminal and npm is useful for the setup chapters. JavaScript and framework experience are optional; the core Sass language can be learned independently.

## How to use this repository

1. Read the lessons in order and try each idea in a small stylesheet.
2. Use the matching Q&A file to review concepts and clarify common mistakes.
3. Follow the matching example and compare the SCSS source with its generated CSS.
4. Build the practical patterns yourself before looking back at the example.
5. Use the summary and keywords files for quick review.

Each roadmap row uses one chapter number and filename across the lessons, Q&A, and examples folders. These links identify planned files; the files will be added chapter by chapter and a link does not mean its content is already present or complete.

## Sass version and project setup

This roadmap follows Dart Sass 1.108.2, the current stable release as of November 26, 2026. Check the official Sass documentation when using version-sensitive features. New module examples use `@use` and `@forward`; Sass `@import` and global built-in function calls are deprecated in Dart Sass starting with version 1.80.0.

Install Sass in a separate practice project with npm:

```bash
npm install --save-dev sass
```

Compile an entry stylesheet and watch source files during development:

```bash
npx sass src/scss/main.scss dist/css/main.css
npx sass --watch src/scss:dist/css
```

For Angular, React, Vite, or another application, use the framework or build tool's documented Sass integration. This repository contains learning notes rather than a runnable Sass project.

## Course roadmap
| No. | Module | Lesson | Q&A | Example |
| ---: | --- | --- | --- | --- |
| 001 | Foundations | [Introduction to Sass and SCSS](lessons/001-introduction-to-sass-and-scss.md) | [Open Q&A](qnas/001-introduction-to-sass-and-scss.md) | [Open Example](examples/001-introduction-to-sass-and-scss.md) |
| 002 | Foundations | [How Sass fits CSS and a web build](lessons/002-how-sass-fits-css-and-a-web-build.md) | [Open Q&A](qnas/002-how-sass-fits-css-and-a-web-build.md) | [Open Example](examples/002-how-sass-fits-css-and-a-web-build.md) |
| 003 | Foundations | [Installing Dart Sass and version basics](lessons/003-installing-dart-sass-and-version-basics.md) | [Open Q&A](qnas/003-installing-dart-sass-and-version-basics.md) | [Open Example](examples/003-installing-dart-sass-and-version-basics.md) |
| 004 | Foundations | [Compiling SCSS to CSS](lessons/004-compiling-scss-to-css.md) | [Open Q&A](qnas/004-compiling-scss-to-css.md) | [Open Example](examples/004-compiling-scss-to-css.md) |
| 005 | Foundations | [SCSS syntax and rules](lessons/005-scss-syntax-and-rules.md) | [Open Q&A](qnas/005-scss-syntax-and-rules.md) | [Open Example](examples/005-scss-syntax-and-rules.md) |
| 006 | Foundations | [Sass variables and scope](lessons/006-sass-variables-and-scope.md) | [Open Q&A](qnas/006-sass-variables-and-scope.md) | [Open Example](examples/006-sass-variables-and-scope.md) |
| 007 | SassScript and values | [Sass value types and null](lessons/007-sass-value-types-and-null.md) | [Open Q&A](qnas/007-sass-value-types-and-null.md) | [Open Example](examples/007-sass-value-types-and-null.md) |
| 008 | SassScript and values | [Numbers, units, and arithmetic](lessons/008-numbers-units-and-arithmetic.md) | [Open Q&A](qnas/008-numbers-units-and-arithmetic.md) | [Open Example](examples/008-numbers-units-and-arithmetic.md) |
| 009 | SassScript and values | [Slash division and calculations](lessons/009-slash-division-and-calculations.md) | [Open Q&A](qnas/009-slash-division-and-calculations.md) | [Open Example](examples/009-slash-division-and-calculations.md) |
| 010 | SassScript and values | [Strings and interpolation](lessons/010-strings-and-interpolation.md) | [Open Q&A](qnas/010-strings-and-interpolation.md) | [Open Example](examples/010-strings-and-interpolation.md) |
| 011 | SassScript and values | [Lists and list functions](lessons/011-lists-and-list-functions.md) | [Open Q&A](qnas/011-lists-and-list-functions.md) | [Open Example](examples/011-lists-and-list-functions.md) |
| 012 | SassScript and values | [Maps and map functions](lessons/012-maps-and-map-functions.md) | [Open Q&A](qnas/012-maps-and-map-functions.md) | [Open Example](examples/012-maps-and-map-functions.md) |
| 013 | SassScript and values | [Colors and the Sass color module](lessons/013-colors-and-sass-color-module.md) | [Open Q&A](qnas/013-colors-and-sass-color-module.md) | [Open Example](examples/013-colors-and-sass-color-module.md) |
| 014 | SassScript and values | [Comments and generated CSS output](lessons/014-comments-and-css-output.md) | [Open Q&A](qnas/014-comments-and-css-output.md) | [Open Example](examples/014-comments-and-css-output.md) |
| 015 | Selectors and reusable logic | [Nesting rules and declarations](lessons/015-nesting-rules-and-declarations.md) | [Open Q&A](qnas/015-nesting-rules-and-declarations.md) | [Open Example](examples/015-nesting-rules-and-declarations.md) |
| 016 | Selectors and reusable logic | [The parent selector](lessons/016-parent-selector.md) | [Open Q&A](qnas/016-parent-selector.md) | [Open Example](examples/016-parent-selector.md) |
| 017 | Selectors and reusable logic | [Interpolation in selectors and property names](lessons/017-interpolation-in-selectors-and-property-names.md) | [Open Q&A](qnas/017-interpolation-in-selectors-and-property-names.md) | [Open Example](examples/017-interpolation-in-selectors-and-property-names.md) |
| 018 | Selectors and reusable logic | [Mixin basics](lessons/018-mixin-basics.md) | [Open Q&A](qnas/018-mixin-basics.md) | [Open Example](examples/018-mixin-basics.md) |
| 019 | Selectors and reusable logic | [Mixin arguments, defaults, and keyword arguments](lessons/019-mixin-arguments-defaults-and-keyword-args.md) | [Open Q&A](qnas/019-mixin-arguments-defaults-and-keyword-args.md) | [Open Example](examples/019-mixin-arguments-defaults-and-keyword-args.md) |
| 020 | Selectors and reusable logic | [Content blocks and context-aware mixins](lessons/020-content-blocks-and-context-aware-mixins.md) | [Open Q&A](qnas/020-content-blocks-and-context-aware-mixins.md) | [Open Example](examples/020-content-blocks-and-context-aware-mixins.md) |
| 021 | Selectors and reusable logic | [Custom functions and return values](lessons/021-custom-functions-and-return-values.md) | [Open Q&A](qnas/021-custom-functions-and-return-values.md) | [Open Example](examples/021-custom-functions-and-return-values.md) |
| 022 | Selectors and reusable logic | [Built-in Sass modules](lessons/022-built-in-sass-modules.md) | [Open Q&A](qnas/022-built-in-sass-modules.md) | [Open Example](examples/022-built-in-sass-modules.md) |
| 023 | Selectors and reusable logic | [Conditionals and Boolean logic](lessons/023-conditionals-and-boolean-logic.md) | [Open Q&A](qnas/023-conditionals-and-boolean-logic.md) | [Open Example](examples/023-conditionals-and-boolean-logic.md) |
| 024 | Selectors and reusable logic | [Loops: each, for, and while](lessons/024-loops-each-for-and-while.md) | [Open Q&A](qnas/024-loops-each-for-and-while.md) | [Open Example](examples/024-loops-each-for-and-while.md) |
| 025 | Selectors and reusable logic | [Generating selectors and utility classes](lessons/025-generating-selectors-and-utilities.md) | [Open Q&A](qnas/025-generating-selectors-and-utilities.md) | [Open Example](examples/025-generating-selectors-and-utilities.md) |
| 026 | Selectors and reusable logic | [@extend and placeholder selectors](lessons/026-extend-placeholder-selectors.md) | [Open Q&A](qnas/026-extend-placeholder-selectors.md) | [Open Example](examples/026-extend-placeholder-selectors.md) |
| 027 | Selectors and reusable logic | [@at-root and nested at-rules](lessons/027-at-root-and-at-rules.md) | [Open Q&A](qnas/027-at-root-and-at-rules.md) | [Open Example](examples/027-at-root-and-at-rules.md) |
| 028 | Modules and migration | [The module system with @use](lessons/028-module-system-with-use.md) | [Open Q&A](qnas/028-module-system-with-use.md) | [Open Example](examples/028-module-system-with-use.md) |
| 029 | Modules and migration | [Module namespaces and private members](lessons/029-module-namespaces-and-private-members.md) | [Open Q&A](qnas/029-module-namespaces-and-private-members.md) | [Open Example](examples/029-module-namespaces-and-private-members.md) |
| 030 | Modules and migration | [Module configuration and !default](lessons/030-module-configuration-and-defaults.md) | [Open Q&A](qnas/030-module-configuration-and-defaults.md) | [Open Example](examples/030-module-configuration-and-defaults.md) |
| 031 | Modules and migration | [Forwarding public APIs with @forward](lessons/031-forwarding-public-apis-with-forward.md) | [Open Q&A](qnas/031-forwarding-public-apis-with-forward.md) | [Open Example](examples/031-forwarding-public-apis-with-forward.md) |
| 032 | Modules and migration | [Partials, load paths, and file naming](lessons/032-partials-load-paths-and-file-naming.md) | [Open Q&A](qnas/032-partials-load-paths-and-file-naming.md) | [Open Example](examples/032-partials-load-paths-and-file-naming.md) |
| 033 | Modules and migration | [Generated CSS and entry points](lessons/033-emitted-css-and-entrypoints.md) | [Open Q&A](qnas/033-emitted-css-and-entrypoints.md) | [Open Example](examples/033-emitted-css-and-entrypoints.md) |
| 034 | Modules and migration | [Legacy @import and global built-in deprecations](lessons/034-legacy-import-and-global-built-in-deprecations.md) | [Open Q&A](qnas/034-legacy-import-and-global-built-in-deprecations.md) | [Open Example](examples/034-legacy-import-and-global-built-in-deprecations.md) |
| 035 | Modules and migration | [Migrating to Sass modules](lessons/035-migrating-to-sass-modules.md) | [Open Q&A](qnas/035-migrating-to-sass-modules.md) | [Open Example](examples/035-migrating-to-sass-modules.md) |
| 036 | CSS integration and design systems | [Sass with CSS at-rules](lessons/036-sass-and-css-at-rules.md) | [Open Q&A](qnas/036-sass-and-css-at-rules.md) | [Open Example](examples/036-sass-and-css-at-rules.md) |
| 037 | CSS integration and design systems | [Sass variables versus CSS custom properties](lessons/037-sass-variables-vs-css-custom-properties.md) | [Open Q&A](qnas/037-sass-variables-vs-css-custom-properties.md) | [Open Example](examples/037-sass-variables-vs-css-custom-properties.md) |
| 038 | CSS integration and design systems | [Runtime theming and design tokens](lessons/038-runtime-theming-and-design-tokens.md) | [Open Q&A](qnas/038-runtime-theming-and-design-tokens.md) | [Open Example](examples/038-runtime-theming-and-design-tokens.md) |
| 039 | CSS integration and design systems | [Reusable responsive mixins](lessons/039-reusable-responsive-mixins.md) | [Open Q&A](qnas/039-reusable-responsive-mixins.md) | [Open Example](examples/039-reusable-responsive-mixins.md) |
| 040 | CSS integration and design systems | [Maps for design systems](lessons/040-maps-for-design-systems.md) | [Open Q&A](qnas/040-maps-for-design-systems.md) | [Open Example](examples/040-maps-for-design-systems.md) |
| 041 | CSS integration and design systems | [Component styles and architecture](lessons/041-component-styles-and-architecture.md) | [Open Q&A](qnas/041-component-styles-and-architecture.md) | [Open Example](examples/041-component-styles-and-architecture.md) |
| 042 | CSS integration and design systems | [Project structure and naming conventions](lessons/042-project-structure-and-naming-conventions.md) | [Open Q&A](qnas/042-project-structure-and-naming-conventions.md) | [Open Example](examples/042-project-structure-and-naming-conventions.md) |
| 043 | Tooling and ecosystem | [npm scripts and build integration](lessons/043-npm-scripts-and-build-integration.md) | [Open Q&A](qnas/043-npm-scripts-and-build-integration.md) | [Open Example](examples/043-npm-scripts-and-build-integration.md) |
| 044 | Tooling and ecosystem | [Sass CLI, watch mode, and build options](lessons/044-sass-cli-watch-and-build-options.md) | [Open Q&A](qnas/044-sass-cli-watch-and-build-options.md) | [Open Example](examples/044-sass-cli-watch-and-build-options.md) |
| 045 | Tooling and ecosystem | [Framework integrations: Angular, React, and Vite](lessons/045-framework-integrations-angular-react-vite.md) | [Open Q&A](qnas/045-framework-integrations-angular-react-vite.md) | [Open Example](examples/045-framework-integrations-angular-react-vite.md) |
| 046 | Tooling and ecosystem | [Authoring a Sass library](lessons/046-authoring-a-sass-library.md) | [Open Q&A](qnas/046-authoring-a-sass-library.md) | [Open Example](examples/046-authoring-a-sass-library.md) |
| 047 | Quality and delivery | [Warnings, errors, and debugging](lessons/047-warnings-errors-and-debugging.md) | [Open Q&A](qnas/047-warnings-errors-and-debugging.md) | [Open Example](examples/047-warnings-errors-and-debugging.md) |
| 048 | Quality and delivery | [Linting and stylesheet quality](lessons/048-linting-and-style-quality.md) | [Open Q&A](qnas/048-linting-and-style-quality.md) | [Open Example](examples/048-linting-and-style-quality.md) |
| 049 | Quality and delivery | [Source maps and browser developer tools](lessons/049-source-maps-and-browser-devtools.md) | [Open Q&A](qnas/049-source-maps-and-browser-devtools.md) | [Open Example](examples/049-source-maps-and-browser-devtools.md) |
| 050 | Quality and delivery | [CSS output, performance, and maintenance](lessons/050-css-output-performance-and-maintenance.md) | [Open Q&A](qnas/050-css-output-performance-and-maintenance.md) | [Open Example](examples/050-css-output-performance-and-maintenance.md) |
| 051 | Quality and delivery | [Accessible motion and responsive patterns](lessons/051-accessible-motion-and-responsive-patterns.md) | [Open Q&A](qnas/051-accessible-motion-and-responsive-patterns.md) | [Open Example](examples/051-accessible-motion-and-responsive-patterns.md) |
| 052 | Quality and delivery | [Browser support and progressive enhancement](lessons/052-browser-support-and-progressive-enhancement.md) | [Open Q&A](qnas/052-browser-support-and-progressive-enhancement.md) | [Open Example](examples/052-browser-support-and-progressive-enhancement.md) |
| 053 | Capstone and reference | [Capstone: build a reusable SCSS design system](lessons/053-capstone-build-a-reusable-scss-design-system.md) | [Open Q&A](qnas/053-capstone-build-a-reusable-scss-design-system.md) | [Open Example](examples/053-capstone-build-a-reusable-scss-design-system.md) |
| 054 | Capstone and reference | [SCSS quick reference and glossary](lessons/054-scss-quick-reference-and-glossary.md) | [Open Q&A](qnas/054-scss-quick-reference-and-glossary.md) | [Open Example](examples/054-scss-quick-reference-and-glossary.md) |
The roadmap lists the 54-chapter course sequence. Every chapter has a lesson, Q&A, and practical example path above.

## Official references

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass syntax](https://sass-lang.com/documentation/syntax/)
- [Sass modules: `@use`](https://sass-lang.com/documentation/at-rules/use/)
- [Sass modules: `@forward`](https://sass-lang.com/documentation/at-rules/forward/)
- [Dart Sass CLI](https://sass-lang.com/documentation/cli/)
- [Sass `@import` deprecation](https://sass-lang.com/documentation/breaking-changes/import/)
