# Linting and stylesheet quality

Linting gives a team repeatable feedback about stylesheet syntax and conventions. It can catch many suspicious or inconsistent patterns, but it cannot decide whether a design is usable, whether a selector matches the intended elements, or whether compiled output behaves correctly in every browser.

A strong SCSS quality workflow combines an SCSS-aware linter, a formatter, clear project conventions, compilation, review, and browser inspection. Each tool has a different job.

## Learning goals

After this lesson, you should be able to:

- Configure Stylelint to parse and lint SCSS.
- Understand the difference between linting, formatting, compilation, and browser validation.
- Choose and document conventions for naming, nesting, declarations, and exceptions.
- Use automatic fixes carefully and review the resulting diff.
- Keep generated files and third-party styles out of the source lint workflow.
- Run formatting, linting, and Sass compilation consistently in local development and CI.

## 1. Define what stylesheet quality means for the project

A quality stylesheet is not simply one that passes every available rule. It should also be understandable, predictable, maintainable, and appropriate for its UI.

Agree on a small set of conventions before tightening a linter:

- File and directory naming.
- Selector and Sass member naming.
- Indentation, whitespace, quote style, and line wrapping.
- Rules for nesting and selector complexity.
- How design tokens and shared mixins are used.
- Which files are authored source and which are generated or vendor code.
- How exceptions are requested and reviewed.

Consistency makes changes easier to review. Rules that encode arbitrary preferences without helping the team understand or maintain the CSS can create noise.

## 2. Use an SCSS-aware linter

Stylelint is a CSS linter that can lint CSS-like languages through a custom syntax and community configurations. For SCSS, the Stylelint project recommends extending the SCSS community configuration <code>stylelint-config-standard-scss</code>. That configuration includes SCSS syntax support and SCSS-specific rules.

Install compatible development dependencies with the package manager used by the project:

~~~sh
npm install --save-dev stylelint stylelint-config-standard-scss
~~~

Check the package requirements and versions together before upgrading. Keep the chosen versions in the project's manifest and lockfile so developers and CI use the same toolchain.

Create <code>stylelint.config.mjs</code> in the project root:

~~~js
/** @type {import('stylelint').Config} */
export default {
  extends: ["stylelint-config-standard-scss"],
  reportDescriptionlessDisables: true,
  reportInvalidScopeDisables: true,
  reportNeedlessDisables: true
};
~~~

The shared config supplies a useful baseline. The reporting options help catch disable comments that lack an explanation, refer to an invalid rule, or no longer suppress a reported problem. Teams can add project-specific rules deliberately after seeing how the baseline behaves on real source files.

If you use a different module format or configuration filename, follow the Stylelint configuration documentation for the installed version.

## 3. Lint authored SCSS files

Run Stylelint on SCSS source files with a quoted glob:

~~~sh
npx stylelint "src/**/*.scss"
~~~

For a project with both SCSS and plain CSS sources, the glob can include both:

~~~sh
npx stylelint "src/**/*.{scss,css}"
~~~

Adapt the paths and extensions to the actual source layout. Avoid including generated output in the same glob as authored Sass. The SCSS shared config uses an SCSS parser; indented <code>.sass</code> files need an appropriate syntax such as <code>postcss-sass</code> configured for those file patterns. Do not assume that parsing <code>.scss</code> also parses indented Sass.

A quoted glob lets Stylelint process the pattern instead of relying on shell-specific glob expansion. A lint error is a signal to inspect the rule, the source, and the project's intent. Fix real issues first. If a rule conflicts with a valid project convention, change the shared config deliberately rather than accumulating local suppressions.

## 4. Keep Stylelint, Prettier, and Sass compilation in their lanes

These tools answer different questions:

| Tool | Main responsibility | It does not prove |
| --- | --- | --- |
| Stylelint | Report configured code-quality and convention problems in stylesheet source. | That Sass compiles or browser behavior is correct. |
| Prettier | Apply consistent formatting to supported files, including SCSS. | That selectors, values, or design choices are correct. |
| Dart Sass | Compile Sass source and report Sass-language errors and deprecations. | That the page looks or behaves as intended in the browser. |
| Browser developer tools | Show loaded stylesheets, matched rules, and computed styles. | That the source follows team conventions or passes CI. |
| Human review | Evaluate intent, readability, design behavior, and trade-offs. | That every file has been mechanically checked. |

A clean lint report does not replace a Sass build. A successful build does not replace visual review. Use each check for the question it can answer.

## 5. Use formatting consistently

Prettier supports SCSS. A shared Prettier configuration and a pinned project dependency help keep editor formatting and CI aligned:

~~~sh
npm install --save-dev prettier
~~~

For example, a team can create <code>.prettierrc.json</code>:

~~~json
{
  "singleQuote": true,
  "printWidth": 100
}
~~~

These values are examples of team preferences. Prettier's <code>printWidth</code> guides wrapping; it is not a strict maximum line-length rule.

Format authored Sass files:

~~~sh
npx prettier --write "src/**/*.scss"
~~~

Check formatting without modifying files, such as in CI:

~~~sh
npx prettier --check "src/**/*.scss"
~~~

Use the same config in editors and command-line runs. Avoid having multiple formatters rewrite the same files with conflicting conventions. If a Stylelint auto-fix and Prettier both change a file, run them in the chosen order and review the final diff.

## 6. Keep nesting and selectors understandable

Sass nesting can express relationships clearly, but deeply nested selectors make the generated CSS more specific and harder to override.

Prefer a shallow structure:

~~~scss
.product-card {
  padding: 1rem;

  &__title {
    margin-block: 0 0.5rem;
  }

  &:hover {
    border-color: var(--color-border-strong);
  }
}
~~~

Avoid mirroring the entire HTML tree in Sass:

~~~scss
/* Fragile: the selector depends on every intermediate element. */
.page {
  .catalog {
    .product-list {
      .product-card {
        .title {
          color: var(--color-text);
        }
      }
    }
  }
}
~~~

A linter can enforce or warn about nesting depth when configured to do so. The right limit depends on the codebase; use it to prompt review, not to treat every nested rule as automatically incorrect.

## 7. Make naming and module conventions predictable

Choose readable naming patterns for Sass variables, mixins, functions, placeholders, and CSS classes. A project may prefer names such as <code>$color-brand</code>, <code>spacing()</code>, and <code>focus-ring</code>, but the important part is that conventions are consistent and descriptive.

For example:

~~~scss
$color-brand: #2457d6;
$space-card: 1rem;

@mixin focus-ring {
  outline: 3px solid var(--color-focus);
  outline-offset: 2px;
}

.card {
  padding: $space-card;

  &:focus-visible {
    @include focus-ring;
  }
}
~~~

Do not make names artificially cryptic just to satisfy a pattern. If the standard configuration's pattern does not fit a documented project convention, configure the relevant rule centrally and explain why.

## 8. Keep lint scopes and ignores explicit

Stylelint ignores <code>node_modules</code> by default. A project may also want to exclude generated bundles, coverage files, or vendored styles. Create a <code>.stylelintignore</code> when these paths should not be linted:

~~~text
dist/
coverage/
vendor/
~~~

Do not ignore authored source merely because it currently produces many lint errors. Bring legacy files under control gradually with an explicit migration plan. If a lint command may match no files, consider whether an empty match should fail: silently linting nothing can make CI appear healthy while missing the source entirely.

For formatting, use <code>.prettierignore</code> for generated or third-party files that should not be rewritten. Keep ignore patterns specific so that a future source file is not excluded by accident.

## 9. Keep rule exceptions narrow and explained

A disable comment should be local to the necessary code, name the rule, and explain the reason:

~~~scss
.vendor-widget {
  /* stylelint-disable-next-line selector-class-pattern -- Required class name from the vendor's markup. */
  &.VendorWidget--active {
    color: var(--color-text-inverse);
  }
}
~~~

Avoid unscoped or file-wide disables when one rule and one selector are the exception. Revisit suppressions when the surrounding code changes. Reporting settings such as <code>reportNeedlessDisables</code> and <code>reportDescriptionlessDisables</code> can help keep exceptions honest.

Do not suppress a rule solely because it caught a genuine issue that is inconvenient to fix. If the rule is not appropriate for the project, change the project configuration and keep the decision visible to everyone.

## 10. Use automatic fixes carefully

Stylelint can automatically fix some reported problems:

~~~sh
npx stylelint "src/**/*.scss" --fix
~~~

Run this on a clean branch or review the entire resulting diff before committing. Automatic fixes are limited to rules that provide a safe fix. They do not make every code-quality decision, and they cannot infer the design intent behind every selector or declaration.

Formatting and lint fixes may touch many files. Separate a broad mechanical cleanup from unrelated feature work when that makes the review clearer.

## 11. Add repeatable project commands

Keep common checks in package scripts so local development and CI use the same commands:

~~~json
{
  "scripts": {
    "lint:scss": "stylelint \"src/**/*.scss\"",
    "format:scss": "prettier --write \"src/**/*.scss\"",
    "format:scss:check": "prettier --check \"src/**/*.scss\"",
    "build:css": "sass src/scss:dist/css"
  }
}
~~~

The directory paths are examples. Use your actual Sass entry points and output directory. A typical CI sequence checks formatting, lints authored files, and compiles Sass. It should fail when a required check fails rather than proceeding to publish incomplete output.

Do not automatically run a write-mode formatter in CI and treat the changed workspace as a pass. Use a check command so differences are reported and can be committed intentionally.

## 12. Review quality beyond the linter

Before merging stylesheet changes, consider:

- Does the selector target the intended elements?
- Does nesting create an unnecessarily specific selector?
- Are shared values expressed through existing design tokens?
- Does the rule respect responsive, focus, hover, and reduced-motion behavior where relevant?
- Can another developer understand why this declaration or exception exists?
- Does compiled CSS contain the intended selectors and values?
- Does the page render correctly in the relevant browsers and interaction states?

A linter can enforce selected patterns; it cannot understand the complete product context. Use code review and browser inspection for the decisions the rules cannot make.

## 13. A practical stylesheet-quality workflow

1. Install and lock compatible Stylelint and SCSS configuration dependencies.
2. Configure one shared SCSS baseline.
3. Point lint and format commands at authored source files.
4. Exclude generated and vendor files explicitly.
5. Run Prettier in write mode when intentionally formatting code.
6. Run Stylelint and inspect each diagnostic.
7. Use auto-fix only for supported changes and review its diff.
8. Compile Sass with the project's normal command.
9. Check the browser for selector matching and computed style behavior.
10. Run the same non-writing checks in CI and require them to pass.

## Key takeaways

- Stylelint can parse and lint SCSS when configured with an SCSS-aware shared configuration.
- Use a formatter for layout, a linter for configured conventions, Sass for compilation, and browser tools for rendered behavior.
- Keep source globs and ignore patterns deliberate so generated files do not pollute checks and authored files are not skipped.
- Use a separate syntax parser for indented <code>.sass</code>; SCSS parsing alone does not cover it.
- Apply auto-fixes carefully, review the diff, and keep rule suppressions narrow and explained.
- CI should run repeatable checks and fail when required lint, format, or compilation steps fail.

## Official references

- [Stylelint: getting started with SCSS](https://stylelint.io/user-guide/get-started/)
- [Stylelint: SCSS standard configuration](https://github.com/stylelint-scss/stylelint-config-standard-scss)
- [Stylelint: configuration](https://stylelint.io/user-guide/configure/)
- [Stylelint: customization and rule exceptions](https://stylelint.io/user-guide/customize/)
- [Stylelint: command-line interface](https://stylelint.io/user-guide/cli/)
- [Stylelint: Sass syntax support](https://stylelint.io/migration-guide/to-14/)
- [PostCSS Sass syntax parser](https://github.com/postcss/postcss-sass)
- [Prettier: options](https://prettier.io/docs/options.html)
- [Prettier: ignoring files](https://prettier.io/docs/ignore.html)

## Next lesson

Continue with [Source maps and browser developer tools](049-source-maps-and-browser-devtools.md).
