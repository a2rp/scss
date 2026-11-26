# Warnings, errors, and debugging - Q&A

This Q&A reviews Sass diagnostics, compiler behavior, and debugging workflows. CLI flags and JavaScript API options can depend on the installed Dart Sass version, so check the documentation for the version your project uses.

## 1. What is the difference between @warn, @error, and @debug?

<code>@warn</code> reports a concern and lets compilation continue. <code>@error</code> stops compilation. <code>@debug</code> prints an expression's value and source location to help investigate behavior.

## 2. When should I use @warn?

Use it when the stylesheet can still produce usable CSS, but the caller should know about a deprecated API, fallback, or suspicious input. Make the warning actionable by explaining what caused it and what to use next.

## 3. When should I use @error?

Use it when the input or state is invalid and continuing would produce misleading or unusable CSS. Include the invalid value and the accepted alternatives so the caller can correct the invocation.

## 4. Does @warn stop Sass compilation?

No. Sass emits the warning and continues compiling. The warning may include a stack trace showing how the code reached the warning.

## 5. Does @error produce a normal successful build?

No. An <code>@error</code> stops compilation and the Sass process reports failure. A CLI may emit an error stylesheet unless configured otherwise, but that output does not make the build successful.

## 6. What information does @debug print?

It prints the expression's value and the location of the <code>@debug</code> statement. Use it to inspect maps, calculations, or values passed to a mixin while diagnosing compilation-time behavior.

## 7. Does @debug show the browser's computed CSS?

No. Sass runs before the browser renders the page. Use <code>@debug</code> for Sass values; use browser developer tools to inspect matched rules and computed styles.

## 8. How do I read a Sass stack trace?

Start with the diagnostic text and source location. Then follow the stack frames to see which function, mixin, or entry point led to that location. The immediate source can be a reusable helper, while a caller in the trace identifies who supplied the problematic input.

## 9. Why can an error point inside a dependency or helper?

Sass reports where it detected the invalid operation. A helper may have received bad input from its caller, or the helper itself may have an incorrect assumption. Follow the trace and decide which boundary owns the fix before editing code.

## 10. What is a Sass deprecation warning?

It signals that Sass encountered a feature or behavior that is deprecated or scheduled to change. Deprecation messages often include an identifier and migration guidance. The identifier can be used with supported CLI options and tooling.

## 11. Is every Sass warning a deprecation warning?

No. A <code>@warn</code> is emitted by your Sass code. A deprecation warning is emitted because Sass encountered a deprecated language feature or behavior. Their source and remediation are different.

## 12. Should I hide all deprecation messages with --quiet?

Usually not. <code>--quiet</code> suppresses warnings and debug output, which can also hide useful messages from your own code. Prefer migrating deprecated code. If suppression is temporarily necessary, target the specific deprecation and document why it remains.

## 13. What does --verbose do?

It asks Dart Sass to print all deprecation warnings instead of limiting repeated output. It is useful for a migration audit where you want to see every reported occurrence.

## 14. What does --quiet-deps do?

It suppresses deprecation warnings from stylesheets Sass classifies as dependencies. It is useful when dependency output obscures application diagnostics, but it does not migrate the dependency and is not a general filter for warnings from your own entry point.

## 15. What does --fatal-deprecation do?

It makes selected deprecations, or all deprecations when supported and configured, cause compilation to fail. Teams can use it in CI after addressing the chosen migration, preventing the deprecated feature from returning unnoticed.

## 16. What does --silence-deprecation do?

It suppresses warnings for specified deprecation identifiers in Sass versions that support the option. Treat it as a temporary migration measure. Record the reason and remove the suppression when the code or dependency has been updated.

## 17. What does --future-deprecation do?

Where supported, it opts into warnings for selected upcoming Sass changes. It lets a project prepare before a change becomes a deprecation. Use only identifiers and behavior supported by the Dart Sass version installed in the project.

## 18. When should I use --trace?

Use <code>--trace</code> when an error's ordinary stack trace lacks enough detail to identify the path through Sass code. It adds diagnostic detail that can help isolate the failing function or mixin.

## 19. What does --no-error-css change?

It tells the Dart Sass CLI not to emit a CSS file describing a compilation error. This can prevent a downstream tool from treating that file as application CSS. The compiler still reports failure; the option does not make the source compile.

## 20. Can error CSS mean the compilation succeeded?

No. A diagnostic stylesheet is not the intended output, and Sass still reports the compilation failure. Check the compiler's exit status and complete terminal output rather than deciding based only on whether a file exists.

## 21. How should CI handle Sass warnings and errors?

CI should preserve the Sass output and honor the process exit code. Errors should fail the build. For deprecations, teams can first audit with verbose output, migrate the reported code, and then make chosen deprecations fatal as a regression guard.

## 22. How should I handle a deprecation from a third-party package?

Identify the package and version in the diagnostic. Check for an upgrade or supported replacement. If an immediate upgrade is not possible, use a narrowly targeted suppression or dependency-warning filter only if supported by the installed Sass version, and track the follow-up migration.

## 23. Why should I avoid blindly replacing slash characters?

A slash may be part of CSS syntax or a Sass division expression. A blanket replacement can change valid CSS. Use the deprecation's location and migration guidance to determine whether the expression is arithmetic and update only the relevant code.

## 24. How do I debug a stylesheet that compiles but looks wrong?

First confirm the page loads the output generated by the current build. Then inspect the affected element's matched rules and computed styles in browser developer tools. Check selector matching, cascade order, specificity, inheritance, media conditions, stale output, and asset paths.

## 25. What are source maps useful for?

When enabled by the build, source maps help browser developer tools navigate from generated CSS back to the Sass source. They help investigate browser-side styling behavior, but they do not replace Sass compiler diagnostics.

## 26. How do I see all warnings in a Sass CLI build?

Check the options supported by the installed CLI with <code>sass --help</code>. For Dart Sass versions that support it, <code>--verbose</code> prints all deprecation warnings. This is especially useful during migration audits.

## 27. Can I control warnings from the Sass JavaScript API?

Yes. The JavaScript API accepts a logger with <code>warn(message, options)</code> and <code>debug(message, options)</code> methods. Warning options include information that can identify a deprecation. Use the API documentation for the installed package version.

## 28. Should a custom Sass logger discard messages?

Usually no. Dropping all warnings and debug messages makes local troubleshooting and CI audits harder. Route them to a visible console or structured build log, and preserve enough source context to locate the message.

## 29. Where should input validation happen in a Sass library?

Validate public inputs at the library boundary where the consumer calls a function or mixin. This gives the caller a clear error close to the incorrect value instead of allowing an obscure failure deeper in an implementation helper.

## 30. What is a good debugging loop for Sass?

Reproduce the issue with the project's Sass version, read the full message and stack, classify the problem, reduce it to a small example, inspect values or computed styles at the correct layer, fix the cause, remove temporary instrumentation, and confirm the compiler's exit status and output.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="046-authoring-a-sass-library.md">046 - Authoring a Sass library - Q&amp;A</a></td><td><a href="../lessons/047-warnings-errors-and-debugging.md">Lesson</a></td><td><a href="../examples/047-warnings-errors-and-debugging.md">Example</a></td><td><a href="048-linting-and-style-quality.md">048 - Linting and stylesheet quality - Q&amp;A</a></td></tr>
  </tbody>
</table>
