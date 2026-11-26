# 032 — Partials, Load Paths, and File Naming

## Learning goals

After this lesson, you should be able to:

- Explain what a Sass partial is and why its filename starts with an underscore.
- Load a partial without writing the underscore or file extension in a module URL.
- Organize entrypoints, reusable modules, and component styles into a predictable directory structure.
- Explain how relative module URLs and configured load paths are resolved.
- Configure a load path with the Dart Sass command line.
- Write portable Sass module URLs that work on Windows, macOS, and Linux.
- Diagnose common missing-file, case, and ambiguous-name problems.

## 1. What a partial is

A Sass partial is a stylesheet intended to be loaded by another stylesheet rather than compiled as a standalone CSS entrypoint. By convention, its filename begins with an underscore.

~~~text
_tokens.scss
_button.scss
_mixins.scss
~~~

The leading underscore tells Sass tools not to compile that file on its own. A partial can still contain variables, mixins, functions, CSS rules, or an entire module. The underscore does not make its members private; it marks the file as a partial.

For example, use a partial for shared design tokens:

~~~scss
// src/scss/_tokens.scss
$brand-color: #1769aa;
$space-small: 0.5rem;
~~~

A separate entrypoint loads those tokens:

~~~scss
// src/scss/app.scss
@use "tokens";

.button {
  background-color: tokens.$brand-color;
  padding: tokens.$space-small;
}
~~~

When Sass resolves the URL tokens, it can find _tokens.scss. The module URL omits both the leading underscore and the file extension.

## 2. Partial names and module URLs

Sass module URLs are not operating-system file paths. They identify a stylesheet using Sass's module resolution rules.

~~~scss
@use "tokens";
@use "components/button";
@forward "utilities";
~~~

These URLs commonly refer to files such as:

~~~text
_tokens.scss
components/_button.scss
_utilities.scss
~~~

You can normally omit the .scss extension, and you can omit the underscore that marks a partial. This keeps module declarations concise and avoids exposing a file's partial marker as part of the public URL.

The import URL should still match the path and spelling of the intended module. Do not assume Sass will ignore capitalization differences or normalize every path variation.

## 3. Choose filenames that communicate purpose

Prefer names that describe what a module provides:

~~~text
_tokens.scss
_typography.scss
_breakpoints.scss
_button.scss
_form-field.scss
_accessible-focus.scss
~~~

Avoid names that reveal little about their role:

~~~text
_stuff.scss
_misc.scss
_file1.scss
~~~

A useful filename helps you recognize a module when it appears in an @use rule or when you search the project. In a larger design system, group related files into directories:

~~~text
src/scss/
├── app.scss
├── tokens/
│   ├── _index.scss
│   ├── _colors.scss
│   └── _spacing.scss
├── components/
│   ├── _button.scss
│   └── _form-field.scss
└── utilities/
    ├── _accessible-focus.scss
    └── _visually-hidden.scss
~~~

Keep application entrypoints distinct from reusable partials. For example, app.scss is compiled into the application stylesheet, while _button.scss is loaded by an entrypoint when its component styles are needed.

## 4. Organize modules around entrypoints

A simple project may have one entrypoint and a few partials:

~~~text
src/scss/
├── app.scss
├── _tokens.scss
├── _typography.scss
└── _buttons.scss
~~~

~~~scss
// src/scss/app.scss
@use "tokens";
@use "typography";
@use "buttons";
~~~

As a project grows, directories can group modules by purpose. Sass supports an _index.scss file as a directory entrypoint:

~~~text
src/scss/
├── app.scss
└── tokens/
    ├── _index.scss
    ├── _colors.scss
    └── _spacing.scss
~~~

~~~scss
// src/scss/tokens/_index.scss
@forward "colors";
@forward "spacing";
~~~

~~~scss
// src/scss/app.scss
@use "tokens" as token;

.card {
  color: token.$text-color;
  padding: token.$space-medium;
}
~~~

When the URL points to the tokens directory, Sass loads its _index.scss. An index file can use or forward modules depending on whether it needs those members locally or wants to expose them to its consumers.

Choose a small number of clear entrypoints. A partial should have a clear owner and purpose, and code should not need many layers of directory traversal to find common modules.

## 5. Relative module URLs

Sass can resolve modules relative to the stylesheet that contains the @use or @forward rule. A relative URL does not need to begin with ./.

~~~text
src/scss/
├── app.scss
├── tokens/
│   └── _colors.scss
└── components/
    └── _button.scss
~~~

~~~scss
// src/scss/components/_button.scss
@use "../tokens/colors";

.button {
  color: colors.$brand-color;
}
~~~

The URL ../tokens/colors is relative to _button.scss. It uses forward slashes and omits the partial underscore and .scss extension.

Relative URLs are useful when a module's dependency is naturally close to it. They also make the relationship between the files explicit. If a module's location changes, its relative URLs may need updating.

## 6. What a load path does

A load path is an additional directory where Sass looks for modules. It is useful when project files or dependencies should be loaded by a short URL from multiple parts of the project.

Suppose the project has:

~~~text
project/
├── src/
│   └── scss/
│       ├── app.scss
│       └── components/
│           └── _button.scss
├── styles/
│   └── _tokens.scss
└── dist/
~~~

The button module can refer to tokens by URL:

~~~scss
// src/scss/components/_button.scss
@use "tokens";

.button {
  color: tokens.$brand-color;
}
~~~

Configure the project styles directory as a load path when compiling:

~~~bash
npx sass --load-path=styles src/scss/app.scss dist/app.css
~~~

Sass can now resolve the URL tokens to styles/_tokens.scss. The load-path directory is a lookup root; it does not change the source file's location.

Load paths can also point into installed dependencies when the package or build tool expects that arrangement. Use the package's documented Sass entrypoint and your build tool's supported integration.

## 7. Resolution order and collisions

Sass first checks for a matching module relative to the stylesheet containing the URL. It checks configured load paths only when no matching relative module is found.

For example:

~~~text
project/
├── src/
│   ├── app.scss
│   └── _tokens.scss
└── shared/
    └── _tokens.scss
~~~

~~~scss
// src/app.scss
@use "tokens";
~~~

If both src and shared are configured as load paths, the relative module src/_tokens.scss is found first because it is beside app.scss. The load-path copy is considered only when a relative match is absent.

When multiple load paths are configured, earlier load paths take precedence over later ones:

~~~bash
npx sass \
  --load-path=styles/project \
  --load-path=styles/shared \
  src/scss/app.scss dist/app.css
~~~

If both configured directories contain a matching module and there is no relative match, Sass checks the earlier load path first. Keep this order deliberate. A duplicate module name can make it unclear which implementation is being used.

To avoid accidental collisions:

- Give project-specific modules descriptive names or directory paths.
- Keep load paths as narrow as practical.
- Avoid placing unrelated modules with identical URLs in different load paths.
- Check local relative files before assuming a load path will win.
- Use the same spelling and capitalization in every module URL.

## 8. Use forward slashes in Sass URLs

Write module URLs with forward slashes, even when working on Windows.

~~~scss
// Portable Sass URL:
@use "components/form-field";

// Avoid Windows path separators in a Sass URL:
@use "components\form-field";
~~~

The Sass URL syntax is designed to behave consistently across operating systems. Backslashes can be interpreted as escapes and can make a stylesheet behave differently than intended.

Use the exact filename capitalization in the URL:

~~~scss
@use "components/form-field";
~~~

If the actual filename is _Form-Field.scss or the directory is Components, do not assume Sass will treat it as the same URL. Sass treats URLs as case-sensitive so that projects behave consistently on case-sensitive systems too.

A URL that works only because a local filesystem ignores letter case may fail in a Linux build or production environment. Match the on-disk spelling and use forward slashes in every @use and @forward URL.

## 9. Partials are still real source modules

The underscore does not mean a partial is ignored when another file loads it.

~~~scss
// src/scss/_badge.scss
.badge {
  border-radius: 999px;
}
~~~

~~~scss
// src/scss/app.scss
@use "badge";
~~~

When app.scss is compiled, Sass includes the CSS from _badge.scss in the result. The partial marker controls standalone compilation behavior; it does not remove the module from the compilation graph.

A module can also contain only Sass members and emit no CSS until a consumer uses them:

~~~scss
// src/scss/_spacing.scss
$space-small: 0.5rem;
$space-medium: 1rem;
~~~

Both are partials. One emits a selector when loaded, while the other only defines variables.

## 10. Entry files and standalone compilation

Files without a leading underscore are commonly used as entrypoints:

~~~text
app.scss
admin.scss
print.scss
~~~

An entrypoint is a stylesheet that a build command compiles into CSS. It can load partials and modules with @use or @forward.

For a one-to-one compilation:

~~~bash
npx sass src/scss/app.scss dist/css/app.css
~~~

For many source stylesheets, Sass can compile a directory into a matching output directory:

~~~bash
npx sass src/scss:dist/css
~~~

In directory mode, partials are not emitted as individual CSS files. Non-partial stylesheets can become outputs, so keep only intentional entrypoints as non-partial Sass files in the source tree. If a file is only meant to be loaded by another module, use the partial naming convention.

Build tools may define their own entrypoints or file-discovery rules. Follow the framework or bundler's Sass integration rather than assuming every tool compiles files exactly like the Sass CLI.

## 11. A complete load-path example

Project structure:

~~~text
project/
├── src/
│   └── scss/
│       ├── app.scss
│       └── components/
│           └── _card.scss
├── styles/
│   ├── _tokens.scss
│   └── _mixins.scss
└── dist/
~~~

Shared modules:

~~~scss
// styles/_tokens.scss
$brand-color: #1769aa;
$text-color: #263238;
$space-medium: 1rem;

// styles/_mixins.scss
@mixin surface {
  border: 1px solid #d5dbe3;
  border-radius: 0.5rem;
}
~~~

Component partial:

~~~scss
// src/scss/components/_card.scss
@use "mixins";
@use "tokens";

.card {
  @include mixins.surface;
  padding: tokens.$space-medium;
  color: tokens.$text-color;
}

.card__title {
  color: tokens.$brand-color;
}
~~~

Application entrypoint:

~~~scss
// src/scss/app.scss
@use "components/card";
~~~

Compile from the project root:

~~~bash
npx sass --load-path=styles src/scss/app.scss dist/app.css
~~~

The component's URLs mixins and tokens resolve through the configured styles load path. The entrypoint loads the component using a path relative to itself.

## 12. Troubleshooting common load errors

### Error: Sass cannot find a stylesheet

Check these items in order:

1. Confirm the file exists and is saved in the expected directory.
2. Confirm the URL is relative to the stylesheet or covered by a configured load path.
3. Confirm you omitted the underscore only in the URL, not from the actual partial filename.
4. Confirm the URL uses forward slashes.
5. Confirm spelling and capitalization match the file and directory names.
6. Confirm the CLI or build configuration includes the intended load path.
7. Check whether another file with the same URL is shadowing the expected one.

### The wrong file loads

Check relative files first. A relative match takes precedence over load paths. If no relative match exists, check the configured load paths in their declared order.

### A partial appears as a separate output

Confirm that its filename begins with an underscore and that the build tool follows Sass partial conventions. Also check whether the build tool has its own file discovery rules.

### A module works on Windows but fails in CI

Look for backslashes in Sass URLs and capitalization differences. A case-insensitive local filesystem can hide a mismatch that a case-sensitive CI filesystem exposes.

### A directory URL does not load what you expect

Check that the directory contains _index.scss or _index.sass and that the URL points to the directory. Check the index file's own @use and @forward URLs as well.

## 13. Naming and organization checklist

Before adding a Sass module, ask:

- Is this an entrypoint or a module loaded by another stylesheet?
- If it is only a module, does its filename start with an underscore?
- Does the name describe the file's contents and role?
- Is it grouped with related files without unnecessary directory depth?
- Does its URL use forward slashes and exact capitalization?
- Is it loaded relative to its consumer, through a clear entrypoint, or from an intentional load path?
- Could another relative file or earlier load path shadow it?
- Is the file expected to emit CSS, or should it only define reusable Sass members?

## 14. Summary

- A partial is conventionally named with a leading underscore and is intended to be loaded by another stylesheet.
- Omit the partial underscore and usually the extension in @use and @forward URLs.
- A partial can still emit CSS when loaded; it is not ignored.
- A directory's _index.scss can act as that directory's module entrypoint.
- Sass resolves a module relative to the current stylesheet before checking load paths.
- Earlier configured load paths take precedence over later ones.
- Use forward slashes and exact case in Sass URLs on every operating system.
- Keep standalone entrypoints distinct from reusable partials.
- Give modules descriptive names and keep load-path resolution predictable.

## Further reading

- [Sass: @use and module resolution](https://sass-lang.com/documentation/at-rules/use/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
