# Linting and stylesheet quality — Examples

This guide builds a small Stylelint and Prettier workflow for authored SCSS. One file intentionally contains a lint problem so you can see a diagnostic and correct it. Do not leave the deliberately broken version in a project that is expected to pass lint.

Package versions and supported options evolve. Install compatible versions, keep them in the project lockfile, and check the current tool documentation when adapting the configuration.

## 1. Create a sample project layout

~~~text
scss-quality/
  package.json
  package-lock.json
  stylelint.config.mjs
  .prettierrc.json
  .stylelintignore
  .prettierignore
  src/
    _tokens.scss
    components/
      card.scss
  playground/
    broken.scss
~~~

The <code>playground/broken.scss</code> file is a temporary practice file. The production lint command below targets <code>src</code>.

## 2. Install compatible development tools

Use npm or the package manager already used by your application:

~~~sh
npm install --save-dev stylelint stylelint-config-standard-scss prettier sass
~~~

Keep the package manifest and lockfile in version control. For reproducible CI, install from the lockfile with the package manager's clean-install command.

The SCSS standard config is community-maintained and includes an SCSS syntax and SCSS-specific rules. Confirm its peer requirements alongside the Stylelint version you install.

## 3. Configure Stylelint

Create <code>stylelint.config.mjs</code> in the project root:

~~~js
/** @type {import('stylelint').Config} */
export default {
  extends: ["stylelint-config-standard-scss"],
  rules: {
    "selector-class-pattern": "^[a-z][a-zA-Z0-9-]*$"
  },
  reportDescriptionlessDisables: true,
  reportInvalidScopeDisables: true,
  reportNeedlessDisables: true
};
~~~

The shared configuration supplies its baseline rules. This project adds a lowercase class-name convention. The reporting options help find disable comments that lack a reason, target an invalid rule, or are no longer needed.

Do not copy a convention without considering your codebase. Adjust the pattern centrally if the project already has a different documented naming scheme.

## 4. Ignore generated and third-party files

Create <code>.stylelintignore</code>:

~~~text
dist/
coverage/
vendor/
~~~

These patterns are examples. Keep generated files out of source linting and exclude vendored code only when the project does not own or maintain it. Do not add the entire authored source directory just to make existing errors disappear.

Prettier can use its own <code>.prettierignore</code> file. For example:

~~~text
dist/
coverage/
vendor/
~~~

Keep both ignore files in sync when they apply to the same generated paths.

## 5. Add formatting preferences

Create <code>.prettierrc.json</code>:

~~~json
{
  "singleQuote": true,
  "printWidth": 100
}
~~~

The options represent team preferences. Prettier's print width guides how it wraps code; it is not a hard limit for every line.

## 6. Define a token module

Create <code>src/_tokens.scss</code>:

~~~scss
$color-brand: #2457d6;
$color-ink: #202534;
$space-md: 1rem;
$radius-card: 0.5rem;
~~~

The leading underscore marks this as a partial intended to be loaded by other Sass files rather than compiled as a standalone entry point.

## 7. Write maintainable component SCSS

Create <code>src/components/card.scss</code>:

~~~scss
@use "../tokens" as tokens;

.card {
  padding: tokens.$space-md;
  border-radius: tokens.$radius-card;
  color: tokens.$color-ink;
  background-color: white;

  &:focus-visible {
    outline: 3px solid tokens.$color-brand;
    outline-offset: 2px;
  }
}
~~~

This example uses a namespace for its Sass members and keeps nesting shallow. The selector targets a class that can be applied to an interactive element; the browser still needs to be checked to confirm focus and other states behave as intended.

## 8. Add repeatable package scripts

Add scripts to <code>package.json</code>:

~~~json
{
  "scripts": {
    "lint:scss": "stylelint \"src/**/*.scss\"",
    "lint:scss:fix": "stylelint \"src/**/*.scss\" --fix",
    "format:scss": "prettier --write \"src/**/*.scss\"",
    "format:scss:check": "prettier --check \"src/**/*.scss\"",
    "build:css": "sass src:dist/css"
  }
}
~~~

The install command writes the compatible package versions into <code>package.json</code> and the lockfile. Merge these scripts into the existing manifest instead of replacing its other fields, and commit the lockfile for reproducible installs.

The Sass command uses <code>src</code> and <code>dist/css</code> as example directories. Sass compiles non-partial Sass files under the input directory, so make sure the source tree contains intentional entry points and the output directory is ignored by version control and linting.

## 9. Run formatting, linting, and compilation

Run the read-only checks:

~~~sh
npm run format:scss:check
npm run lint:scss
npm run build:css
~~~

Use the write-mode formatter when you deliberately want to format source:

~~~sh
npm run format:scss
~~~

Stylelint can also fix supported issues:

~~~sh
npm run lint:scss:fix
~~~

After either write-mode command, inspect the diff. Auto-fixes can update only the issues that the configured rules know how to fix; they do not decide whether the CSS behavior is correct.

## 10. Observe and fix a lint error

Create a temporary <code>playground/broken.scss</code>:

~~~scss
.card {
  colour: #fff;
  padding: 0px;
}
~~~

Lint that one file directly:

~~~sh
npx stylelint playground/broken.scss
~~~

The unknown property is an intentional practice error. Stylelint should report it with the file and rule information. The exact formatting and rule messages depend on the installed config and version.

Correct the property and remove the unit from zero where the active rules require it:

~~~scss
.card {
  color: #fff;
  padding: 0;
}
~~~

Run Stylelint again:

~~~sh
npx stylelint playground/broken.scss
~~~

Once the exercise is complete, remove the temporary file or move a corrected, intentional example into the source tree.

## 11. Add a narrow exception for a vendor class

Suppose third-party markup requires a class that conflicts with this project's lowercase naming rule. Keep the exception close to the selector and explain it:

~~~scss
/* stylelint-disable-next-line selector-class-pattern -- Vendor markup requires this class name. */
.VendorWidget {
  color: #202534;
}
~~~

Run the rule-disable reports during cleanup if a comment is no longer needed:

~~~sh
npx stylelint "src/**/*.scss" --report-needless-disables
~~~

An exception should document an external constraint, not conceal an issue that the project can fix.

## 12. Check the effective Stylelint configuration

If a file appears not to follow the expected rules, print the configuration Stylelint resolves for that file:

~~~sh
npx stylelint src/components/card.scss --print-config
~~~

This can reveal whether the expected shared config, overrides, and rules are active. The printed options can be extensive, so focus on the relevant rule and syntax settings.

## 13. Add a basic CI sequence

A CI job can run the same read-only commands as local development:

~~~sh
npm ci
npm run format:scss:check
npm run lint:scss
npm run build:css
~~~

The exact clean-install command depends on the package manager. Ensure each check's failure status stops the job. Do not run formatting with <code>--write</code> in CI and treat modified files as a pass; use the check command so the required formatting change can be reviewed and committed.

## 14. Choose the correct tool for each report

| Report or symptom | First response |
| --- | --- |
| Unknown property or malformed stylesheet syntax | Check the declaration and Stylelint parser message. |
| A team naming rule reports a valid exception | Confirm the convention, then adjust the shared config or add a narrow documented disable. |
| Formatting check fails | Run Prettier locally, review the diff, and commit the formatting change. |
| Stylelint passes but Sass compilation fails | Read the Dart Sass diagnostic; linting does not compile the project. |
| Sass compiles but the page is wrong | Inspect loaded CSS, matched selectors, and computed styles in browser tools. |
| Generated files appear in lint output | Narrow the input glob and update ignore patterns. |

## Practice checklist

- Install the tools in a project and commit the lockfile.
- Configure the SCSS shared rules and the team's class naming convention.
- Run Stylelint on the temporary broken example and fix the reported property.
- Run Prettier check and write commands, then compare their effects.
- Add a documented, rule-specific exception and remove it after the constraint ends.
- Compile Sass separately and inspect the browser output.
- Confirm CI fails if any required check fails.

## Official references

- [Stylelint: getting started with SCSS](https://stylelint.io/user-guide/get-started/)
- [Stylelint: SCSS standard configuration](https://github.com/stylelint-scss/stylelint-config-standard-scss)
- [Stylelint: configuration and overrides](https://stylelint.io/user-guide/configure/)
- [Stylelint: command-line options and autofix](https://stylelint.io/user-guide/cli/)
- [Stylelint: customization and rule exceptions](https://stylelint.io/user-guide/customize/)
- [Stylelint: errors and warnings](https://stylelint.io/user-guide/errors/)
- [PostCSS SCSS parser](https://github.com/postcss/postcss-scss)
- [Prettier: options](https://prettier.io/docs/options.html)
- [Prettier: ignoring files](https://prettier.io/docs/ignore.html)
