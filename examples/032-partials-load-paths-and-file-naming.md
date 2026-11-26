# 032 — Partials, Load Paths, and File Naming: Examples

Each example shows a small file layout and the module URLs Sass should use. Paths in Sass rules use forward slashes. CLI commands are intended to run from the project root unless stated otherwise.

## 1. Load a token partial from an entrypoint

### Files

~~~text
project/
└── src/
    └── scss/
        ├── app.scss
        └── _tokens.scss
~~~

~~~scss
// src/scss/_tokens.scss
$brand-color: #1769aa;
$space-small: 0.5rem;
~~~

~~~scss
// src/scss/app.scss
@use "tokens" as token;

.button {
  color: token.$brand-color;
  padding: token.$space-small;
}
~~~

Compile the entrypoint:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

Expected CSS:

~~~css
.button {
  color: #1769aa;
  padding: 0.5rem;
}
~~~

Sass resolves tokens to _tokens.scss. The underscore and extension are left out of the URL.

## 2. A loaded partial can emit CSS

A partial is not skipped when another stylesheet loads it.

~~~text
src/scss/
├── app.scss
└── _badge.scss
~~~

~~~scss
// src/scss/_badge.scss
.badge {
  display: inline-block;
  border-radius: 999px;
  padding: 0.2rem 0.6rem;
}
~~~

~~~scss
// src/scss/app.scss
@use "badge";
~~~

Compile app.scss and the emitted CSS includes the .badge rule. The leading underscore means this file is a partial, not that its CSS is discarded.

## 3. Resolve a module with a relative URL

### Files

~~~text
src/scss/
├── tokens/
│   └── _colors.scss
└── components/
    └── _notice.scss
~~~

~~~scss
// src/scss/tokens/_colors.scss
$warning: #8a4b00;
$warning-surface: #fff4d6;
~~~

~~~scss
// src/scss/components/_notice.scss
@use "../tokens/colors";

.notice {
  color: colors.$warning;
  background-color: colors.$warning-surface;
}
~~~

The URL is relative to _notice.scss. It uses forward slashes and omits the underscore and extension.

## 4. Load a directory entrypoint

### Files

~~~text
src/scss/
├── app.scss
└── tokens/
    ├── _index.scss
    ├── _colors.scss
    └── _spacing.scss
~~~

~~~scss
// src/scss/tokens/_colors.scss
$text-color: #263238;
$brand-color: #1769aa;

// src/scss/tokens/_spacing.scss
$space-small: 0.5rem;
$space-medium: 1rem;

// src/scss/tokens/_index.scss
@forward "colors";
@forward "spacing";

// src/scss/app.scss
@use "tokens" as token;

.card {
  color: token.$text-color;
  border-color: token.$brand-color;
  padding: token.$space-medium;
}
~~~

Sass resolves the URL tokens through tokens/_index.scss. That index forwards the public members from the color and spacing modules.

## 5. Resolve a shared module through a load path

Here the component imports a common token module by its short URL.

### Files

~~~text
project/
├── src/
│   └── scss/
│       ├── app.scss
│       └── components/
│           └── _card.scss
├── styles/
│   └── _tokens.scss
└── dist/
~~~

~~~scss
// styles/_tokens.scss
$surface: #ffffff;
$text: #263238;
$border: #d5dbe3;

// src/scss/components/_card.scss
@use "tokens";

.card {
  background-color: tokens.$surface;
  color: tokens.$text;
  border: 1px solid tokens.$border;
}

// src/scss/app.scss
@use "components/card";
~~~

Compile with styles as a load path:

~~~bash
npx sass --load-path=styles src/scss/app.scss dist/app.css
~~~

The component's relative directory does not contain a matching tokens module, so Sass checks the configured load path and finds styles/_tokens.scss.

## 6. A relative match takes priority over a load path

### Files

~~~text
project/
├── src/
│   └── scss/
│       ├── components/
│       │   ├── _card.scss
│       │   └── _tokens.scss
│       └── app.scss
└── shared/
    └── _tokens.scss
~~~

~~~scss
// src/scss/components/_tokens.scss
$source-name: "local component tokens";
$accent: #1769aa;

// shared/_tokens.scss
$source-name: "shared tokens";
$accent: #8a4b00;

// src/scss/components/_card.scss
@use "tokens";

.card::before {
  content: tokens.$source-name;
  border-color: tokens.$accent;
}

// src/scss/app.scss
@use "components/card";
~~~

~~~bash
npx sass --load-path=shared src/scss/app.scss dist/app.css
~~~

The module URL in _card.scss resolves first to the nearby src/scss/components/_tokens.scss. The same URL in shared is only used if no matching relative file exists.

If this was accidental, rename one module or make the intended directory relationship explicit so the chosen source is clear.

## 7. Earlier load paths take precedence

Two configured load paths can contain a module with the same URL:

~~~text
project/
├── src/scss/
│   └── app.scss
├── styles/
│   ├── brand/
│   │   └── _tokens.scss
│   └── shared/
│       └── _tokens.scss
└── dist/
~~~

~~~scss
// styles/brand/_tokens.scss
$brand-color: #1769aa;

// styles/shared/_tokens.scss
$brand-color: #5a6573;

// src/scss/app.scss
@use "tokens";

.site-title {
  color: tokens.$brand-color;
}
~~~

~~~bash
npx sass \
  --load-path=styles/brand \
  --load-path=styles/shared \
  src/scss/app.scss dist/app.css
~~~

Because there is no matching relative module, Sass checks styles/brand first and resolves tokens to that directory's _tokens.scss. If the two files are meant to be distinct, use distinct URLs or namespaces rather than relying on load-path order.

## 8. Keep Sass URLs portable

Correct Sass URL:

~~~scss
@use "components/form-field";
~~~

Avoid Windows separators in a Sass URL:

~~~scss
@use "components\form-field";
~~~

Also match the file's capitalization exactly. If the actual path is components/_form-field.scss, use components/form-field. Do not rely on a case-insensitive local filesystem to fix a mismatch.

The filesystem paths passed to the shell and the URLs written inside Sass files are different things. Use the CLI's documented path syntax for the operating system, and use forward slashes in Sass URLs.

## 9. Compile a directory without emitting partials

### Files

~~~text
src/scss/
├── app.scss
├── _tokens.scss
└── components/
    └── _card.scss
~~~

~~~scss
// src/scss/_tokens.scss
$surface: #ffffff;

// src/scss/components/_card.scss
@use "../tokens";

.card {
  background-color: tokens.$surface;
}

// src/scss/app.scss
@use "components/card";
~~~

Compile the source directory:

~~~bash
npx sass src/scss:dist/css
~~~

The non-partial app.scss is an entrypoint and can produce dist/css/app.css. The underscore-prefixed _tokens.scss and _card.scss are loaded as modules and are not emitted as independent CSS files.

Keep non-partial source files in this directory only when they are intended as compilation inputs. If you use a bundler, check its own entrypoint discovery rules too.

## 10. A complete project with organized modules

### Files

~~~text
project/
├── src/
│   └── scss/
│       ├── app.scss
│       ├── tokens/
│       │   ├── _index.scss
│       │   ├── _colors.scss
│       │   └── _spacing.scss
│       └── components/
│           ├── _card.scss
│           └── _button.scss
└── dist/
~~~

~~~scss
// src/scss/tokens/_colors.scss
$brand: #1769aa;
$text: #263238;
$surface: #ffffff;

// src/scss/tokens/_spacing.scss
$space-small: 0.5rem;
$space-medium: 1rem;

// src/scss/tokens/_index.scss
@forward "colors";
@forward "spacing";

// src/scss/components/_card.scss
@use "../tokens" as token;

.card {
  padding: token.$space-medium;
  color: token.$text;
  background-color: token.$surface;
  border: 1px solid #d5dbe3;
  border-radius: 0.5rem;
}

// src/scss/components/_button.scss
@use "../tokens" as token;

.button {
  padding: token.$space-small token.$space-medium;
  border: 0;
  background-color: token.$brand;
  color: white;
}

// src/scss/app.scss
@use "components/card";
@use "components/button";
~~~

Compile the application entrypoint:

~~~bash
npx sass src/scss/app.scss dist/app.css
~~~

The organization makes the roles clear:

- app.scss is compiled directly.
- Partial modules are loaded with @use.
- The tokens directory has an _index.scss entrypoint.
- Relative URLs describe dependencies between nearby source files.
- All Sass module URLs use forward slashes.

## 11. Troubleshoot a missing module

Suppose the file is:

~~~text
src/scss/components/_form-field.scss
~~~

This URL is correct from a stylesheet in src/scss:

~~~scss
@use "components/form-field";
~~~

Common incorrect versions include:

~~~scss
// The underscore belongs to the filename, not normally the URL.
@use "components/_form-field";

// This path uses Windows separators instead of Sass URL separators.
@use "components\form-field";

// This capitalization must match the actual path exactly.
@use "Components/Form-Field";
~~~

Check the relative directory of the stylesheet containing the rule. If the intended module is not relative to it, either write the correct relative URL or add a deliberate load path.

## 12. Practice tasks

1. Create _colors.scss and _spacing.scss partials, then load them from app.scss.
2. Move both files into a tokens directory and add an _index.scss entrypoint.
3. Create a component partial in a nested directory and load tokens with a relative URL.
4. Move the shared tokens to a separate directory and configure that directory as a load path.
5. Put a same-named token module beside the component and in the load path. Predict which one Sass resolves.
6. Compile a source directory and identify which files become CSS outputs and which remain partial modules.
7. Change one URL's capitalization and explain why another operating system may expose the mismatch.

## Quick reference

~~~text
Partial file:       _tokens.scss
Module URL:         @use "tokens";

Nested partial:     components/_button.scss
Module URL:         @use "components/button";

Directory entry:    tokens/_index.scss
Module URL:         @use "tokens";
~~~

~~~bash
npx sass src/scss/app.scss dist/app.css
npx sass --load-path=src/scss src/scss/app.scss dist/app.css
npx sass --load-path=styles/project --load-path=styles/shared src/scss/app.scss dist/app.css
npx sass src/scss:dist/css
~~~

## Related material

- [Lesson: Partials, load paths, and file naming](../lessons/032-partials-load-paths-and-file-naming.md)
- [Q&A: Partials, load paths, and file naming](../qnas/032-partials-load-paths-and-file-naming.md)
- [Sass: @use and module resolution](https://sass-lang.com/documentation/at-rules/use/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
