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

Every roadmap chapter has a corresponding lesson, Q&A, and example file. The numbered filenames make it easy to match the three resources for each topic.

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
<table width="100%">
  <thead>
    <tr>
      <th width="6%" align="right">No.</th>
      
      <th width="68%">Lesson</th>
      <th width="13%">Q&amp;A</th>
      <th width="13%">Example</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td align="right"><code>001</code></td>
      
      <td><a href="lessons/001-introduction-to-sass-and-scss.md">Introduction to Sass and SCSS</a></td>
      <td><a href="qnas/001-introduction-to-sass-and-scss.md">Open Q&amp;A</a></td>
      <td><a href="examples/001-introduction-to-sass-and-scss.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>002</code></td>
      
      <td><a href="lessons/002-how-sass-fits-css-and-a-web-build.md">How Sass fits CSS and a web build</a></td>
      <td><a href="qnas/002-how-sass-fits-css-and-a-web-build.md">Open Q&amp;A</a></td>
      <td><a href="examples/002-how-sass-fits-css-and-a-web-build.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>003</code></td>
      
      <td><a href="lessons/003-installing-dart-sass-and-version-basics.md">Installing Dart Sass and version basics</a></td>
      <td><a href="qnas/003-installing-dart-sass-and-version-basics.md">Open Q&amp;A</a></td>
      <td><a href="examples/003-installing-dart-sass-and-version-basics.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>004</code></td>
      
      <td><a href="lessons/004-compiling-scss-to-css.md">Compiling SCSS to CSS</a></td>
      <td><a href="qnas/004-compiling-scss-to-css.md">Open Q&amp;A</a></td>
      <td><a href="examples/004-compiling-scss-to-css.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>005</code></td>
      
      <td><a href="lessons/005-scss-syntax-and-rules.md">SCSS syntax and rules</a></td>
      <td><a href="qnas/005-scss-syntax-and-rules.md">Open Q&amp;A</a></td>
      <td><a href="examples/005-scss-syntax-and-rules.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>006</code></td>
      
      <td><a href="lessons/006-sass-variables-and-scope.md">Sass variables and scope</a></td>
      <td><a href="qnas/006-sass-variables-and-scope.md">Open Q&amp;A</a></td>
      <td><a href="examples/006-sass-variables-and-scope.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>007</code></td>
      
      <td><a href="lessons/007-sass-value-types-and-null.md">Sass value types and null</a></td>
      <td><a href="qnas/007-sass-value-types-and-null.md">Open Q&amp;A</a></td>
      <td><a href="examples/007-sass-value-types-and-null.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>008</code></td>
      
      <td><a href="lessons/008-numbers-units-and-arithmetic.md">Numbers, units, and arithmetic</a></td>
      <td><a href="qnas/008-numbers-units-and-arithmetic.md">Open Q&amp;A</a></td>
      <td><a href="examples/008-numbers-units-and-arithmetic.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>009</code></td>
      
      <td><a href="lessons/009-slash-division-and-calculations.md">Slash division and calculations</a></td>
      <td><a href="qnas/009-slash-division-and-calculations.md">Open Q&amp;A</a></td>
      <td><a href="examples/009-slash-division-and-calculations.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>010</code></td>
      
      <td><a href="lessons/010-strings-and-interpolation.md">Strings and interpolation</a></td>
      <td><a href="qnas/010-strings-and-interpolation.md">Open Q&amp;A</a></td>
      <td><a href="examples/010-strings-and-interpolation.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>011</code></td>
      
      <td><a href="lessons/011-lists-and-list-functions.md">Lists and list functions</a></td>
      <td><a href="qnas/011-lists-and-list-functions.md">Open Q&amp;A</a></td>
      <td><a href="examples/011-lists-and-list-functions.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>012</code></td>
      
      <td><a href="lessons/012-maps-and-map-functions.md">Maps and map functions</a></td>
      <td><a href="qnas/012-maps-and-map-functions.md">Open Q&amp;A</a></td>
      <td><a href="examples/012-maps-and-map-functions.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>013</code></td>
      
      <td><a href="lessons/013-colors-and-sass-color-module.md">Colors and the Sass color module</a></td>
      <td><a href="qnas/013-colors-and-sass-color-module.md">Open Q&amp;A</a></td>
      <td><a href="examples/013-colors-and-sass-color-module.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>014</code></td>
      
      <td><a href="lessons/014-comments-and-css-output.md">Comments and generated CSS output</a></td>
      <td><a href="qnas/014-comments-and-css-output.md">Open Q&amp;A</a></td>
      <td><a href="examples/014-comments-and-css-output.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>015</code></td>
      
      <td><a href="lessons/015-nesting-rules-and-declarations.md">Nesting rules and declarations</a></td>
      <td><a href="qnas/015-nesting-rules-and-declarations.md">Open Q&amp;A</a></td>
      <td><a href="examples/015-nesting-rules-and-declarations.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>016</code></td>
      
      <td><a href="lessons/016-parent-selector.md">The parent selector</a></td>
      <td><a href="qnas/016-parent-selector.md">Open Q&amp;A</a></td>
      <td><a href="examples/016-parent-selector.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>017</code></td>
      
      <td><a href="lessons/017-interpolation-in-selectors-and-property-names.md">Interpolation in selectors and property names</a></td>
      <td><a href="qnas/017-interpolation-in-selectors-and-property-names.md">Open Q&amp;A</a></td>
      <td><a href="examples/017-interpolation-in-selectors-and-property-names.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>018</code></td>
      
      <td><a href="lessons/018-mixin-basics.md">Mixin basics</a></td>
      <td><a href="qnas/018-mixin-basics.md">Open Q&amp;A</a></td>
      <td><a href="examples/018-mixin-basics.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>019</code></td>
      
      <td><a href="lessons/019-mixin-arguments-defaults-and-keyword-args.md">Mixin arguments, defaults, and keyword arguments</a></td>
      <td><a href="qnas/019-mixin-arguments-defaults-and-keyword-args.md">Open Q&amp;A</a></td>
      <td><a href="examples/019-mixin-arguments-defaults-and-keyword-args.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>020</code></td>
      
      <td><a href="lessons/020-content-blocks-and-context-aware-mixins.md">Content blocks and context-aware mixins</a></td>
      <td><a href="qnas/020-content-blocks-and-context-aware-mixins.md">Open Q&amp;A</a></td>
      <td><a href="examples/020-content-blocks-and-context-aware-mixins.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>021</code></td>
      
      <td><a href="lessons/021-custom-functions-and-return-values.md">Custom functions and return values</a></td>
      <td><a href="qnas/021-custom-functions-and-return-values.md">Open Q&amp;A</a></td>
      <td><a href="examples/021-custom-functions-and-return-values.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>022</code></td>
      
      <td><a href="lessons/022-built-in-sass-modules.md">Built-in Sass modules</a></td>
      <td><a href="qnas/022-built-in-sass-modules.md">Open Q&amp;A</a></td>
      <td><a href="examples/022-built-in-sass-modules.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>023</code></td>
      
      <td><a href="lessons/023-conditionals-and-boolean-logic.md">Conditionals and Boolean logic</a></td>
      <td><a href="qnas/023-conditionals-and-boolean-logic.md">Open Q&amp;A</a></td>
      <td><a href="examples/023-conditionals-and-boolean-logic.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>024</code></td>
      
      <td><a href="lessons/024-loops-each-for-and-while.md">Loops: each, for, and while</a></td>
      <td><a href="qnas/024-loops-each-for-and-while.md">Open Q&amp;A</a></td>
      <td><a href="examples/024-loops-each-for-and-while.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>025</code></td>
      
      <td><a href="lessons/025-generating-selectors-and-utilities.md">Generating selectors and utility classes</a></td>
      <td><a href="qnas/025-generating-selectors-and-utilities.md">Open Q&amp;A</a></td>
      <td><a href="examples/025-generating-selectors-and-utilities.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>026</code></td>
      
      <td><a href="lessons/026-extend-placeholder-selectors.md">@extend and placeholder selectors</a></td>
      <td><a href="qnas/026-extend-placeholder-selectors.md">Open Q&amp;A</a></td>
      <td><a href="examples/026-extend-placeholder-selectors.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>027</code></td>
      
      <td><a href="lessons/027-at-root-and-at-rules.md">@at-root and nested at-rules</a></td>
      <td><a href="qnas/027-at-root-and-at-rules.md">Open Q&amp;A</a></td>
      <td><a href="examples/027-at-root-and-at-rules.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>028</code></td>
      
      <td><a href="lessons/028-module-system-with-use.md">The module system with @use</a></td>
      <td><a href="qnas/028-module-system-with-use.md">Open Q&amp;A</a></td>
      <td><a href="examples/028-module-system-with-use.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>029</code></td>
      
      <td><a href="lessons/029-module-namespaces-and-private-members.md">Module namespaces and private members</a></td>
      <td><a href="qnas/029-module-namespaces-and-private-members.md">Open Q&amp;A</a></td>
      <td><a href="examples/029-module-namespaces-and-private-members.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>030</code></td>
      
      <td><a href="lessons/030-module-configuration-and-defaults.md">Module configuration and !default</a></td>
      <td><a href="qnas/030-module-configuration-and-defaults.md">Open Q&amp;A</a></td>
      <td><a href="examples/030-module-configuration-and-defaults.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>031</code></td>
      
      <td><a href="lessons/031-forwarding-public-apis-with-forward.md">Forwarding public APIs with @forward</a></td>
      <td><a href="qnas/031-forwarding-public-apis-with-forward.md">Open Q&amp;A</a></td>
      <td><a href="examples/031-forwarding-public-apis-with-forward.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>032</code></td>
      
      <td><a href="lessons/032-partials-load-paths-and-file-naming.md">Partials, load paths, and file naming</a></td>
      <td><a href="qnas/032-partials-load-paths-and-file-naming.md">Open Q&amp;A</a></td>
      <td><a href="examples/032-partials-load-paths-and-file-naming.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>033</code></td>
      
      <td><a href="lessons/033-emitted-css-and-entrypoints.md">Generated CSS and entry points</a></td>
      <td><a href="qnas/033-emitted-css-and-entrypoints.md">Open Q&amp;A</a></td>
      <td><a href="examples/033-emitted-css-and-entrypoints.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>034</code></td>
      
      <td><a href="lessons/034-legacy-import-and-global-built-in-deprecations.md">Legacy @import and global built-in deprecations</a></td>
      <td><a href="qnas/034-legacy-import-and-global-built-in-deprecations.md">Open Q&amp;A</a></td>
      <td><a href="examples/034-legacy-import-and-global-built-in-deprecations.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>035</code></td>
      
      <td><a href="lessons/035-migrating-to-sass-modules.md">Migrating to Sass modules</a></td>
      <td><a href="qnas/035-migrating-to-sass-modules.md">Open Q&amp;A</a></td>
      <td><a href="examples/035-migrating-to-sass-modules.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>036</code></td>
      
      <td><a href="lessons/036-sass-and-css-at-rules.md">Sass with CSS at-rules</a></td>
      <td><a href="qnas/036-sass-and-css-at-rules.md">Open Q&amp;A</a></td>
      <td><a href="examples/036-sass-and-css-at-rules.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>037</code></td>
      
      <td><a href="lessons/037-sass-variables-vs-css-custom-properties.md">Sass variables versus CSS custom properties</a></td>
      <td><a href="qnas/037-sass-variables-vs-css-custom-properties.md">Open Q&amp;A</a></td>
      <td><a href="examples/037-sass-variables-vs-css-custom-properties.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>038</code></td>
      
      <td><a href="lessons/038-runtime-theming-and-design-tokens.md">Runtime theming and design tokens</a></td>
      <td><a href="qnas/038-runtime-theming-and-design-tokens.md">Open Q&amp;A</a></td>
      <td><a href="examples/038-runtime-theming-and-design-tokens.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>039</code></td>
      
      <td><a href="lessons/039-reusable-responsive-mixins.md">Reusable responsive mixins</a></td>
      <td><a href="qnas/039-reusable-responsive-mixins.md">Open Q&amp;A</a></td>
      <td><a href="examples/039-reusable-responsive-mixins.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>040</code></td>
      
      <td><a href="lessons/040-maps-for-design-systems.md">Maps for design systems</a></td>
      <td><a href="qnas/040-maps-for-design-systems.md">Open Q&amp;A</a></td>
      <td><a href="examples/040-maps-for-design-systems.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>041</code></td>
      
      <td><a href="lessons/041-component-styles-and-architecture.md">Component styles and architecture</a></td>
      <td><a href="qnas/041-component-styles-and-architecture.md">Open Q&amp;A</a></td>
      <td><a href="examples/041-component-styles-and-architecture.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>042</code></td>
      
      <td><a href="lessons/042-project-structure-and-naming-conventions.md">Project structure and naming conventions</a></td>
      <td><a href="qnas/042-project-structure-and-naming-conventions.md">Open Q&amp;A</a></td>
      <td><a href="examples/042-project-structure-and-naming-conventions.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>043</code></td>
      
      <td><a href="lessons/043-npm-scripts-and-build-integration.md">npm scripts and build integration</a></td>
      <td><a href="qnas/043-npm-scripts-and-build-integration.md">Open Q&amp;A</a></td>
      <td><a href="examples/043-npm-scripts-and-build-integration.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>044</code></td>
      
      <td><a href="lessons/044-sass-cli-watch-and-build-options.md">Sass CLI, watch mode, and build options</a></td>
      <td><a href="qnas/044-sass-cli-watch-and-build-options.md">Open Q&amp;A</a></td>
      <td><a href="examples/044-sass-cli-watch-and-build-options.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>045</code></td>
      
      <td><a href="lessons/045-framework-integrations-angular-react-vite.md">Framework integrations: Angular, React, and Vite</a></td>
      <td><a href="qnas/045-framework-integrations-angular-react-vite.md">Open Q&amp;A</a></td>
      <td><a href="examples/045-framework-integrations-angular-react-vite.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>046</code></td>
      
      <td><a href="lessons/046-authoring-a-sass-library.md">Authoring a Sass library</a></td>
      <td><a href="qnas/046-authoring-a-sass-library.md">Open Q&amp;A</a></td>
      <td><a href="examples/046-authoring-a-sass-library.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>047</code></td>
      
      <td><a href="lessons/047-warnings-errors-and-debugging.md">Warnings, errors, and debugging</a></td>
      <td><a href="qnas/047-warnings-errors-and-debugging.md">Open Q&amp;A</a></td>
      <td><a href="examples/047-warnings-errors-and-debugging.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>048</code></td>
      
      <td><a href="lessons/048-linting-and-style-quality.md">Linting and stylesheet quality</a></td>
      <td><a href="qnas/048-linting-and-style-quality.md">Open Q&amp;A</a></td>
      <td><a href="examples/048-linting-and-style-quality.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>049</code></td>
      
      <td><a href="lessons/049-source-maps-and-browser-devtools.md">Source maps and browser developer tools</a></td>
      <td><a href="qnas/049-source-maps-and-browser-devtools.md">Open Q&amp;A</a></td>
      <td><a href="examples/049-source-maps-and-browser-devtools.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>050</code></td>
      
      <td><a href="lessons/050-css-output-performance-and-maintenance.md">CSS output, performance, and maintenance</a></td>
      <td><a href="qnas/050-css-output-performance-and-maintenance.md">Open Q&amp;A</a></td>
      <td><a href="examples/050-css-output-performance-and-maintenance.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>051</code></td>
      
      <td><a href="lessons/051-accessible-motion-and-responsive-patterns.md">Accessible motion and responsive patterns</a></td>
      <td><a href="qnas/051-accessible-motion-and-responsive-patterns.md">Open Q&amp;A</a></td>
      <td><a href="examples/051-accessible-motion-and-responsive-patterns.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>052</code></td>
      
      <td><a href="lessons/052-browser-support-and-progressive-enhancement.md">Browser support and progressive enhancement</a></td>
      <td><a href="qnas/052-browser-support-and-progressive-enhancement.md">Open Q&amp;A</a></td>
      <td><a href="examples/052-browser-support-and-progressive-enhancement.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>053</code></td>
      
      <td><a href="lessons/053-capstone-build-a-reusable-scss-design-system.md">Capstone: build a reusable SCSS design system</a></td>
      <td><a href="qnas/053-capstone-build-a-reusable-scss-design-system.md">Open Q&amp;A</a></td>
      <td><a href="examples/053-capstone-build-a-reusable-scss-design-system.md">Open Example</a></td>
    </tr>
    <tr>
      <td align="right"><code>054</code></td>
      
      <td><a href="lessons/054-scss-quick-reference-and-glossary.md">SCSS quick reference and glossary</a></td>
      <td><a href="qnas/054-scss-quick-reference-and-glossary.md">Open Q&amp;A</a></td>
      <td><a href="examples/054-scss-quick-reference-and-glossary.md">Open Example</a></td>
    </tr>
  </tbody>
</table>
The roadmap lists the 54-chapter course sequence. Every chapter has a lesson, Q&A, and practical example path above.
