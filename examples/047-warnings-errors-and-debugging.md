# Warnings, errors, and debugging — Examples

These examples demonstrate Sass diagnostics as separate, focused exercises. The <code>@error</code> example is intentionally invalid and should fail; run it separately from the examples expected to compile successfully.

Use the Dart Sass version configured by your project. CLI options and deprecation identifiers can vary by version, so confirm support with <code>sass --help</code> and the version's documentation.

## 1. Create a small practice workspace

~~~text
sass-diagnostics/
  src/
    debug-values.scss
    warning.scss
    error.scss
    deprecation-audit.scss
    styles.scss
  scripts/
    compile.mjs
  dist/
~~~

Install or use Dart Sass in the project. For example, when using npm:

~~~sh
npm install --save-dev sass
~~~

From the project root, the CLI can compile one file like this:

~~~sh
npx sass src/warning.scss dist/warning.css
~~~

## 2. Emit a recoverable warning

Create <code>src/warning.scss</code>:

~~~scss
@mixin legacy-card($padding) {
  @warn "legacy-card() is deprecated; use the card() mixin instead.";

  padding: $padding;
}

.card {
  @include legacy-card(1rem);
}
~~~

Compile it:

~~~sh
npx sass src/warning.scss dist/warning.css
~~~

Sass prints a warning and continues. The resulting CSS still contains the <code>.card</code> rule and its padding. A warning does not itself fail compilation.

A warning should help the caller take action. If this is a real migration, name the replacement API accurately and remove the compatibility warning once callers have moved.

## 3. Stop compilation for invalid input

Create <code>src/error.scss</code>:

~~~scss
@use "sass:map";

$spacing: (
  compact: 0.5rem,
  regular: 1rem,
  spacious: 1.5rem
);

@function space($name) {
  @if not map.has-key($spacing, $name) {
    @error "Unknown spacing token: #{$name}. Choose compact, regular, or spacious.";
  }

  @return map.get($spacing, $name);
}

.panel {
  padding: space(regular);
}

.invalid-panel {
  padding: space(giant);
}
~~~

Run this file separately:

~~~sh
npx sass src/error.scss dist/error.css
~~~

The <code>space(giant)</code> call triggers <code>@error</code>. Sass reports the failure and a call trace; this is an expected failure for the exercise. Do not interpret an error CSS artifact, if your CLI creates one, as a successful application stylesheet.

After confirming the failure, change <code>giant</code> to a key in the map and compile again to see the successful path.

## 4. Print values while debugging

Create <code>src/debug-values.scss</code>:

~~~scss
@use "sass:map";

$breakpoints: (
  small: 40rem,
  medium: 64rem,
  large: 80rem
);

$base-size: 1rem;
$scale: 1.25;
$heading-size: $base-size * $scale * $scale;

@debug $breakpoints;
@debug map.get($breakpoints, medium);
@debug $heading-size;

.page-title {
  font-size: $heading-size;
}
~~~

Compile:

~~~sh
npx sass src/debug-values.scss dist/debug-values.css
~~~

The terminal shows the values and the source locations of the <code>@debug</code> statements. The stylesheet still compiles. Remove temporary debug lines after you have found the cause, unless the message is intentionally maintained as part of the project.

## 5. Inspect mixin input and provide a fallback

This mixin warns for an unsupported visual option but can use a safe fallback. Save it as <code>src/styles.scss</code>:

~~~scss
@mixin surface($tone) {
  @if $tone == brand {
    background-color: #2457d6;
    color: white;
  } @else if $tone == neutral {
    background-color: #f2f4f8;
    color: #202534;
  } @else {
    @warn "Unknown surface tone: #{$tone}. Using the neutral surface.";
    background-color: #f2f4f8;
    color: #202534;
  }
}

.card {
  @include surface(neutral);
}

.notice {
  @include surface(mystery);
}
~~~

Compile it:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

The <code>.notice</code> call emits a warning but uses the neutral surface declarations. If the fallback is not safe for the API, replace the warning with an <code>@error</code> so invalid input cannot silently produce misleading output.

## 6. See a deprecation without hiding other messages

Deprecations depend on the Dart Sass version and the feature used. A legacy slash-division expression is one feature that may produce a deprecation notice in Dart Sass versions that still report it:

~~~scss
.legacy-size {
  width: (12px / 2);
}
~~~

For new arithmetic, use the module function:

~~~scss
@use "sass:math";

.modern-size {
  width: math.div(12px, 2);
}
~~~

CSS slash syntax such as <code>font: 16px/1.5 sans-serif</code> is a different use of slash. Inspect the reported source and Sass migration guidance instead of replacing every slash mechanically.

To audit deprecations during compilation, use the supported verbose flag:

~~~sh
npx sass --verbose src/deprecation-audit.scss dist/deprecation-audit.css
~~~

To make one known deprecation fail in a migration-focused CI job, use the identifier recognized by the installed Sass version:

~~~sh
npx sass --fatal-deprecation=slash-div src/deprecation-audit.scss dist/deprecation-audit.css
~~~

If the project cannot migrate immediately, a targeted suppression is possible in versions that support it:

~~~sh
npx sass --silence-deprecation=slash-div src/deprecation-audit.scss dist/deprecation-audit.css
~~~

These are migration controls, not permanent cleanup. Do not combine them with <code>--quiet</code> just to hide all warnings. If dependency deprecations drown out application messages, check whether <code>--quiet-deps</code> is supported and appropriate for the project's dependency policy.

## 7. Keep the build failed when Sass fails

A CI shell command should preserve Sass's exit status. For example:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

If Sass reports an error, the command returns a failure status. A build script should propagate that status so later packaging or deployment steps do not publish a broken stylesheet.

When an error stylesheet could confuse downstream tools, disable it where supported:

~~~sh
npx sass --no-error-css src/error.scss dist/error.css
~~~

This command is expected to fail because <code>src/error.scss</code> intentionally calls <code>space(giant)</code>. The option prevents error CSS output; it does not suppress the error or change the failure status.

For more diagnostic detail, run:

~~~sh
npx sass --trace src/error.scss dist/error.css
~~~

## 8. Format diagnostics from the JavaScript API

Create <code>scripts/compile.mjs</code>:

~~~js
import * as sass from "sass";

const result = sass.compile("src/styles.scss", {
  logger: {
    warn(message, options) {
      const kind = options.deprecation ? "deprecation" : "warning";
      console.warn("[Sass " + kind + "] " + message);
    },
    debug(message) {
      console.debug("[Sass debug] " + message);
    }
  }
});

console.log(result.css);
~~~

Run the script:

~~~sh
node scripts/compile.mjs
~~~

The logger makes Sass warnings and debug messages visible with a label. A Sass compilation error is thrown by the compile call; if the build script catches it for custom formatting, it should report the failure and exit with a nonzero status rather than continuing as if compilation succeeded. Check the JavaScript API documentation for the installed <code>sass</code> version before relying on logger option details.

## 9. Keep a temporary debug line easy to remove

When inspecting a function argument, place the debug statement next to the relevant logic:

~~~scss
@function rem-from-px($pixels, $root-size: 16px) {
  @debug "pixels: #{$pixels}; root size: #{$root-size}";

  @return $pixels / $root-size * 1rem;
}
~~~

If Dart Sass reports a division deprecation here, treat it as an arithmetic migration task and use the correct module function:

~~~scss
@use "sass:math";

@function rem-from-px($pixels, $root-size: 16px) {
  @return math.div($pixels, $root-size) * 1rem;
}
~~~

In actual code, declare <code>@use</code> rules at the top of the stylesheet before other rules. This example separates the original debugging point from the corrected arithmetic to make the migration visible.

## 10. Choose the correct tool for the symptom

| Symptom | First tool to use | What to inspect |
| --- | --- | --- |
| Sass stops with an error | Sass CLI output; add <code>--trace</code> if needed | First error, file and line, call trace, input values |
| Sass warns but writes CSS | Sass CLI output | Warning source and whether the fallback is acceptable |
| A deprecation appears repeatedly | <code>--verbose</code> during audit | Feature identifier, source files, dependency ownership |
| Build output exists after an error | Process exit status and CLI diagnostics | Whether it is error CSS rather than usable output |
| CSS compiles but the page is styled incorrectly | Browser developer tools | Loaded file, matched rules, computed styles, cascade |
| A value passed to a mixin seems wrong | Temporary <code>@debug</code> | Value at the call boundary and inside the helper |

## Practice checklist

- Compile <code>warning.scss</code> and observe that Sass continues.
- Compile <code>error.scss</code> and observe that the process fails.
- Compile <code>debug-values.scss</code> and find the reported source locations.
- Use verbose output to audit deprecations supported by your Sass version.
- Inspect the generated CSS and the command's exit status separately.
- Remove temporary debug statements and temporary deprecation suppressions.
- For browser-only styling issues, inspect computed styles in the browser.

## Official references

- [Sass: @warn](https://sass-lang.com/documentation/at-rules/warn/)
- [Sass: @error](https://sass-lang.com/documentation/at-rules/error/)
- [Sass: @debug](https://sass-lang.com/documentation/at-rules/debug/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass deprecations](https://sass-lang.com/documentation/breaking-changes/)
- [Sass JavaScript API logger](https://sass-lang.com/documentation/js-api/interfaces/logger/)
