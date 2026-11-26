# Warnings, errors, and debugging

Sass diagnostics help explain what happened while a stylesheet was compiled. A useful workflow distinguishes intentional messages from compilation failures, locates the source of a problem, and fixes the cause instead of hiding every message.

This lesson covers Sass's <code>@warn</code>, <code>@error</code>, and <code>@debug</code> at-rules; compiler and deprecation diagnostics; Dart Sass command-line controls; and a repeatable way to investigate failures.

## Learning goals

After this lesson, you should be able to:

- Choose between <code>@warn</code>, <code>@error</code>, and <code>@debug</code>.
- Read a Sass diagnostic and trace it back to the relevant source.
- Distinguish a Sass warning from a deprecation notice and a compilation error.
- Use Dart Sass CLI flags to reveal, filter, or deliberately fail on diagnostics.
- Debug build and watch workflows without mistaking generated error CSS for a successful build.
- Route Sass messages through a JavaScript logger when compiling programmatically.

## 1. Know the three Sass diagnostic at-rules

Each diagnostic rule communicates a different condition.

| Rule | Purpose | Does compilation continue? |
| --- | --- | --- |
| <code>@warn</code> | Report a condition that may need attention while output can still be generated. | Yes |
| <code>@error</code> | Reject an invalid state that should not produce a usable stylesheet. | No |
| <code>@debug</code> | Print a value and the source location for temporary investigation. | Yes |

### Use @warn for recoverable or deprecated input

A warning tells the stylesheet author that something deserves attention, but Sass can keep compiling:

~~~scss
@mixin legacy-card($padding) {
  @warn "legacy-card() is deprecated; use the card() mixin instead.";

  padding: $padding;
}
~~~

Warnings are especially useful while maintaining a compatibility layer. State what triggered the warning and what the caller should do next. Avoid repeating the same warning from every low-level helper when one message at the public API boundary is enough.

Sass prints the warning with a stack trace that helps identify where the call came from. A warning does not automatically mean the generated CSS is wrong, and suppressing it does not make the underlying issue disappear.

### Use @error for invalid input

An error stops Sass compilation. Use it when continuing would generate misleading or invalid CSS:

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
~~~

A clear error explains the invalid value and how to provide a valid one. This is more useful than allowing a missing map lookup or a malformed value to cause a confusing failure later.

Do not use <code>@error</code> for a condition from which your API can safely recover. It prevents the current compilation from producing a successful stylesheet.

### Use @debug while investigating values

<code>@debug</code> prints an expression's value and the source location where it appears:

~~~scss
$columns: 12;
$gap: 1.25rem;

@debug $columns;
@debug $gap;
@debug $columns * $gap;
~~~

Debug output is useful for checking a map, calculation, or value passed into a mixin. Remove temporary debug statements once the issue is understood, or keep only those that provide a deliberate benefit to maintainers.

Debugging a Sass value does not show the browser's computed style. Sass runs during compilation; inspect computed styles in browser developer tools when the problem appears only at runtime or depends on layout, the cascade, or DOM state.

## 2. Read diagnostics from the source outward

When Sass reports a warning or error:

1. Read the first message carefully. It often states the invalid construct or deprecated feature.
2. Note the filename and line number where Sass detected it.
3. Read the nearby code and identify the value or rule involved.
4. Follow the stack trace upward to find the mixin, function, or entry point that supplied it.
5. Determine whether the source is your application, a shared internal module, or a dependency.
6. Fix the cause, then compile again and check that the diagnostic is gone for the intended reason.

The location where Sass detects a problem may be inside a reusable function, while the stack trace shows which caller supplied the bad value. Fix the correct boundary: sometimes the caller has an invalid input; sometimes the helper has a faulty assumption.

Keep the full diagnostic when sharing a build failure. The message, source path, line, and stack are often all needed to reproduce it. Truncating CI output to only the final line can hide the useful cause.

## 3. Separate warnings from deprecation notices

A custom <code>@warn</code> message is created by your Sass code. A deprecation warning is generated because Sass encountered a language feature or behavior that is scheduled for removal or has changed.

Deprecation messages commonly include a deprecation identifier and migration guidance. The identifier lets you target a known migration in CLI options or tooling. Consult the current Sass deprecation documentation because identifiers, timelines, and supported options can change between Dart Sass releases.

Common migration work may include replacing legacy module loading, updating built-in function usage, or reviewing division expressions. Follow the message and official migration guidance rather than performing a blind search-and-replace: a slash can be CSS syntax as well as an arithmetic expression.

Treat deprecations as planned maintenance. A sensible order is:

1. Identify which deprecated feature and source file are reported.
2. Determine whether the file is maintained by your project or comes from a dependency.
3. Upgrade or migrate the responsible code.
4. Add a temporary, narrowly scoped suppression only when migration cannot happen immediately.
5. Remove that suppression after the code is fixed.

## 4. Use Dart Sass CLI options intentionally

The Dart Sass CLI has options for controlling diagnostic output and deprecations. Check <code>sass --help</code> for the exact options supported by the version installed in your project.

| Option | Use |
| --- | --- |
| <code>--verbose</code> | Print all deprecation warnings rather than limiting repeated output. Useful during audits. |
| <code>--quiet</code> | Silence warnings and debug messages. Avoid using it as a permanent substitute for fixing issues. |
| <code>--quiet-deps</code> | Silence deprecation warnings from files Sass classifies as dependencies. It does not hide every warning from your own entry point. |
| <code>--fatal-deprecation</code> | Treat selected deprecations, or all deprecations when supported by the CLI, as compilation failures. Useful in CI after migration. |
| <code>--silence-deprecation</code> | Suppress selected deprecation identifiers temporarily while a migration is pending. |
| <code>--future-deprecation</code> | Opt in to warnings for selected upcoming changes when the installed Sass release supports them. |
| <code>--trace</code> | Include a more detailed stack trace for errors. |
| <code>--no-error-css</code> | Do not emit a CSS error stylesheet when compilation fails. |

For example, an audit can reveal repeated notices and a CI job can reject a specific deprecated feature:

~~~sh
sass --verbose src/scss:dist/css
sass --fatal-deprecation=slash-div src/scss:dist/css
~~~

These commands illustrate the options; the identifier must match a deprecation supported by the Dart Sass version in use. A command that treats one deprecation as fatal is a migration guard, not a general replacement for reading errors.

Avoid adding <code>--quiet</code> to a shared build script simply to make output look clean. It may hide newly introduced warnings and debugging messages from your team. If a dependency is noisy, determine whether an upgrade is available; use <code>--quiet-deps</code> only when filtering dependency deprecations is an intentional trade-off.

## 5. Understand failed builds and error CSS

A failed compilation is still a failed compilation even if the CLI emits CSS describing the error. Error CSS can be useful in some development workflows, but it is not the intended application stylesheet and must not be mistaken for a successful artifact.

Use <code>--no-error-css</code> when a generated error stylesheet would confuse a downstream tool or deployment process. In every environment, rely on the compiler's exit status as well as the output file. CI should stop when Sass exits with an error.

When a watch command is running, distinguish a successful rebuild from a failed rebuild in its terminal output. After fixing the source, wait for a successful compile and verify the generated stylesheet again.

## 6. Debug the right layer

Sass diagnostics answer questions about compilation. The browser answers questions about the result after HTML, CSS, assets, and runtime state are combined.

### If Sass fails to compile

- Check the first error and its source location.
- Verify that modules resolve from the intended load paths.
- Check required arguments and values passed into functions or mixins.
- Inspect the call stack and reproduce with the smallest relevant entry point.
- Run with <code>--trace</code> when the ordinary trace does not show enough context.

### If compilation succeeds but the page looks wrong

- Confirm the page loaded the stylesheet you just compiled.
- Inspect the element's computed styles and matched rules in browser developer tools.
- Check selector matching, cascade order, specificity, inheritance, and media conditions.
- Look for stale build output, incorrect asset paths, or another stylesheet overriding the rule.
- Use source maps when the project generates them to navigate from compiled CSS to Sass source.

Do not add more Sass logging when the actual problem is a browser cascade conflict. Likewise, browser inspection cannot explain why a Sass function rejected an argument before any stylesheet was generated.

## 7. Route messages from the Sass JavaScript API

When compiling with the Sass JavaScript API, a custom logger can format or route warnings and debug messages. The warning options distinguish deprecation messages from other Sass warnings:

~~~js
import * as sass from "sass";

const result = sass.compile("src/styles/main.scss", {
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

Keep custom logger behavior observable. A logger that discards every message can make local development and CI harder to diagnose. For a build pipeline, you can route warnings to structured logs, include the source span when available, and apply project-specific policies to deprecations.

The API and logger options depend on the Sass package version. Check the API reference for the version in your lockfile before relying on a newer option.

## 8. Make diagnostics useful and maintainable

Good diagnostics are concise and actionable:

- Name the invalid or deprecated input.
- Include the accepted alternatives or the next step.
- Report the issue at the layer where the caller can fix it.
- Avoid leaking secrets or huge internal values into logs.
- Avoid emitting the same warning repeatedly from nested helpers.
- Keep CI output and failure status available to developers.
- Remove temporary <code>@debug</code> statements after investigation.

For reusable Sass APIs, validate public inputs at the boundary. This gives consumers an error that points to their call instead of a confusing downstream failure.

## 9. A repeatable debugging workflow

Use this short loop when Sass output is unexpected:

1. Reproduce the issue with the same Sass version and command used by the project.
2. Read the complete diagnostic, including file, line, and stack.
3. Classify it as a custom warning, deprecation, Sass error, build-tool error, or browser-only issue.
4. Reduce the input to the smallest stylesheet that still shows the problem.
5. Inspect relevant values with temporary <code>@debug</code> statements, or inspect computed styles in the browser if compilation succeeds.
6. Fix the source or dependency, then remove temporary instrumentation.
7. Re-run the project build and confirm that success is reflected in both the process exit status and generated output.
8. If a deprecation cannot yet be fixed, document a narrowly targeted temporary suppression and its removal plan.

## Key takeaways

- <code>@warn</code> reports a recoverable concern, <code>@error</code> stops compilation, and <code>@debug</code> prints a value and source location.
- Read a diagnostic's source location and call stack before changing code.
- A Sass deprecation is different from a custom warning; migrate the feature instead of hiding all output.
- CLI flags such as <code>--verbose</code>, <code>--quiet-deps</code>, and <code>--fatal-deprecation</code> serve different purposes.
- Error CSS does not turn a failed build into a successful one.
- Use Sass tools for compilation problems and browser developer tools for computed-style problems.

## Official references

- [Sass: @warn](https://sass-lang.com/documentation/at-rules/warn/)
- [Sass: @error](https://sass-lang.com/documentation/at-rules/error/)
- [Sass: @debug](https://sass-lang.com/documentation/at-rules/debug/)
- [Dart Sass command-line interface](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass deprecations](https://sass-lang.com/documentation/breaking-changes/)
- [Sass JavaScript API logger](https://sass-lang.com/documentation/js-api/interfaces/logger/)

## Next lesson

Continue with [Linting and style quality](048-linting-and-style-quality.md).
