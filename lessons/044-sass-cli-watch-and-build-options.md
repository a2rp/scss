# Sass CLI, watch mode, and build options

Dart Sass’s command-line interface (CLI) compiles Sass files to CSS and offers options for mapping inputs to outputs, watching changes, choosing CSS formatting, generating source maps, and diagnosing problems.

This lesson focuses on the CLI itself. For npm script conventions, see the previous lesson. The commands below assume the <code>sass</code> executable is available, for example through a project-local Sass dependency.

## Learning goals

After this lesson, you should be able to:

- Choose one-to-one or many-to-many compilation.
- Map entry files or source directories to CSS outputs.
- Use watch and update modes while understanding their limits.
- Configure CSS output style, source maps, and module search paths.
- Diagnose compiler errors and warnings with appropriate CLI flags.
- Select options deliberately for development, CI, and release output.

## 1. Check the Sass executable

Check which Sass version the current environment runs:

~~~sh
sass --version
~~~

When Sass is installed as an npm project dependency, use <code>npx sass --version</code> or run it through an npm script. The project-local executable helps keep the CLI behavior aligned with the version in the lockfile.

Dart Sass is the primary Sass implementation. Check the official CLI documentation when choosing an option because flags and compatibility can depend on the Dart Sass version.

## 2. One-to-one mode

One-to-one mode compiles a single source file to one output:

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

If the output path is omitted, compiled CSS is printed to standard output:

~~~sh
sass src/styles/app.scss
~~~

Use an explicit output file for a build. The input should usually be an application entry point that loads its partials and dependencies.

## 3. Many-to-many mode

Many-to-many mode maps one or more inputs to outputs:

~~~sh
sass src/styles/public.scss:public/css/public.css src/styles/admin.scss:public/css/admin.css
~~~

Use this when several entry files need separate output files. Each entry should have a clear purpose; do not map every reusable partial to a standalone output.

## 4. Directory compilation

A directory can be mapped to an output directory:

~~~sh
sass src/themes:public/css/themes
~~~

Sass compiles Sass files in the source tree to corresponding CSS files. It ignores partials whose names begin with an underscore, so partials can hold shared code without becoming separate outputs.

Choose directory mode only when the non-partial files are meant to be compiled independently. If the project has a small known set of entry points, explicit file mappings are easier to review.

## 5. Watch mode

Use <code>--watch</code> to compile initially and keep running as source files change:

~~~sh
sass --watch src/styles:public/css
~~~

The process stays open. Stop it with Ctrl+C. You can also watch one entry file and write to its corresponding output:

~~~sh
sass --watch src/styles/app.scss:public/css/app.css
~~~

Watch mode acts like an initial update and then recompiles when watched source files change. Use it for development; use a one-time build command in CI or deployment.

## 6. What Sass watches

Sass watches directories passed directly on the command line, parent directories of filenames passed on the command line, and configured load paths. It does not add arbitrary folders to its watch set just because an <code>@use</code> dependency refers to a file there.

If a changed dependency is outside the watched source tree, add an appropriate source directory or load path to the watch invocation. When the operating system does not report file changes reliably, consider <code>--poll</code>.

## 7. Polling mode

<code>--poll</code> must be used with <code>--watch</code>. It makes Sass periodically check for file changes instead of relying on operating-system notifications:

~~~sh
sass --watch --poll src/styles:public/css
~~~

Polling may help with some remote or virtual filesystems. It uses periodic checks, so prefer normal watch notifications when they work.

## 8. Update only changed files

The <code>--update</code> option enables many-to-many mode and compiles outputs whose sources or dependencies have changed since the CSS was generated:

~~~sh
sass --update src/styles:public/css
~~~

Sass reports which outputs it updates. This can save work in a directory with multiple entry files. It is not a replacement for dependency installation or the project’s overall build graph.

## 9. Stop after the first error

When compiling multiple inputs, Sass normally attempts to continue with other inputs after an error. Use <code>--stop-on-error</code> when the command should stop after the first compilation error:

~~~sh
sass --stop-on-error src/themes:public/css/themes
~~~

This can make CI logs easier to interpret when one failure is enough to fail the build.

## 10. Expanded and compressed CSS

Dart Sass supports two CSS output styles:

~~~sh
sass --style=expanded src/styles/app.scss public/css/app.css
sass --style=compressed src/styles/app.scss public/css/app.css
~~~

Expanded output is the default and is easier to inspect. Compressed output removes formatting whitespace and writes compact CSS. This affects output formatting, not Sass source semantics.

## 11. Source maps

Dart Sass generates source maps by default when it writes CSS. The map links generated CSS back to Sass source for browser debugging.

Disable maps for a chosen output with:

~~~sh
sass --no-source-map src/styles/app.scss public/css/app.css
~~~

Use relative source-map URLs when the output and source tree travel together. Absolute file URLs point to a location on the machine that compiled them and generally are not useful after deployment.

## 12. Embed source content or the map

<code>--embed-sources</code> embeds the contents of contributing Sass files in the source map. This can make the map much larger.

<code>--embed-source-map</code> embeds the map data in the CSS instead of writing a separate map file. These options change the size and portability of debugging artifacts; use them only when the workflow needs that behavior.

## 13. Add Sass load paths

A load path is an additional directory Sass searches for module URLs:

~~~sh
sass --load-path=src/shared src/styles/app.scss public/css/app.css
~~~

You can pass more than one load path:

~~~sh
sass --load-path=src/shared --load-path=node_modules src/styles/app.scss public/css/app.css
~~~

Earlier load paths take precedence over later ones. Keep relative module URLs as the default when they express the dependency clearly; add load paths when they solve a real resolution need.

## 14. Use the Node package importer when appropriate

Dart Sass supports an optional Node package importer through <code>--pkg-importer=node</code> in supported versions:

~~~sh
sass --pkg-importer=node src/styles/app.scss public/css/app.css
~~~

This enables Sass to resolve supported Node package URLs. Check the installed Sass CLI’s version and package documentation before using it in a shared project.

## 15. Control charset output

By default, when a stylesheet contains non-ASCII characters, Sass may emit a charset declaration in expanded output or a UTF-8 byte-order mark in compressed output. Use <code>--no-charset</code> to suppress this:

~~~sh
sass --no-charset src/styles/app.scss public/css/app.css
~~~

Use this only when the project’s encoding and delivery requirements support omitting the marker.

## 16. Error CSS

When Sass writes to a file, error CSS is enabled by default. If compilation fails, Sass can write CSS containing a visible diagnostic so the problem may be seen in a browser. The process still reports the compilation error.

Use <code>--no-error-css</code> to disable diagnostic CSS:

~~~sh
sass --no-error-css src/styles/app.scss public/css/app.css
~~~

With watch or update mode, disabling error CSS causes Sass to delete CSS output files when a compilation error occurs. Consider whether that behavior is appropriate for the project.

## 17. Warnings, quiet output, and diagnostics

Use <code>--verbose</code> to show all deprecation warnings:

~~~sh
sass --verbose src/styles/app.scss public/css/app.css
~~~

Use <code>--quiet-deps</code> to suppress deprecation warnings from dependencies loaded through load paths, while keeping warnings from first-party code visible:

~~~sh
sass --load-path=node_modules --quiet-deps src/styles/app.scss public/css/app.css
~~~

<code>--quiet</code> silences warnings and debug output broadly. Avoid using it as a permanent fix for warnings in your own source.

## 18. Treat selected deprecations as errors

<code>--fatal-deprecation</code> can turn a selected deprecation into a compilation error:

~~~sh
sass --fatal-deprecation=import src/styles/app.scss public/css/app.css
~~~

This can help teams catch deprecated features before a future upgrade. Only use deprecation IDs supported by the installed Sass version, and introduce this option after reviewing what the codebase and dependencies currently use.

## 19. Add terminal color and stack traces when debugging

The CLI normally chooses terminal color based on whether the output appears to support it. You can set it explicitly:

~~~sh
sass --color src/styles/app.scss public/css/app.css
sass --no-color src/styles/app.scss public/css/app.css
~~~

For compiler debugging, <code>--trace</code> prints a full stack trace:

~~~sh
sass --trace src/styles/app.scss public/css/app.css
~~~

A stack trace is most useful when a normal error message does not explain the failure.

## 20. Read SCSS from standard input

Use <code>--stdin</code> when another command supplies Sass source on standard input:

~~~sh
sass --stdin public/css/generated.css
~~~

Without a file extension, Sass treats standard input as SCSS unless the <code>--indented</code> option is passed. Standard input cannot be combined with many-to-many mode.

For normal project code, compiling a named entry file is easier to trace and debug.

## 21. Print help and inspect supported options

Use the executable’s own help output:

~~~sh
sass --help
~~~

This is useful when working with a different Sass installation or checking whether the installed version supports a particular flag. A package-local CLI can be queried with <code>npx sass --help</code>.

## 22. Choose useful option combinations

Development watcher:

~~~sh
sass --watch --style=expanded src/styles/app.scss:public/css/app.css
~~~

Release build without a source map:

~~~sh
sass --style=compressed --no-source-map src/styles/app.scss public/css/app.css
~~~

CI compilation that stops at the first error and fails on a selected deprecation:

~~~sh
sass --stop-on-error --fatal-deprecation=import src/styles/app.scss public/css/app.css
~~~

Use the smallest set of flags that serves the build goal. A long command is harder to review and maintain.

## 23. Common mistakes

### Watching the wrong folders

A module dependency outside the watched directories may not trigger recompilation. Add the relevant source location or load path to the watch command.

### Using directory mode for partials as entry points

Partials are ignored as standalone outputs. Compile the actual entry file or directory of non-partial entry files.

### Leaving watch mode in a build pipeline

A watcher intentionally stays open. Use a one-time compile command for a completed build.

### Assuming compressed CSS fixes expensive styles

Compression removes formatting characters; it does not simplify selectors or change the design. Review the CSS itself if performance or output size is a concern.

### Hiding deprecation warnings

Warnings can signal upcoming incompatibilities. Fix first-party warnings and suppress dependency warnings only when there is a clear reason.

### Embedding source maps without a need

Embedded sources and maps can make assets substantially larger. Use these options for a specific debugging or distribution requirement.

## Quick reference

| Goal | CLI option or form |
| --- | --- |
| Compile one file | <code>sass input.scss output.css</code> |
| Map multiple files | <code>input.scss:output.css</code> pairs |
| Map a source directory | <code>sass input/:output/</code> |
| Watch for edits | <code>--watch</code> |
| Poll for edits | <code>--watch --poll</code> |
| Compile changed outputs | <code>--update</code> |
| Stop after first error | <code>--stop-on-error</code> |
| Compact output | <code>--style=compressed</code> |
| Disable source maps | <code>--no-source-map</code> |
| Add a module search path | <code>--load-path=path</code> |
| Show all deprecation warnings | <code>--verbose</code> |
| Suppress dependency deprecations | <code>--quiet-deps</code> |
| Show command options | <code>--help</code> |

## Further reading

- [Dart Sass CLI](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass CLI overview](https://sass-lang.com/documentation/cli/)
- [Sass installation](https://sass-lang.com/install/)
- [Sass breaking changes and deprecations](https://sass-lang.com/documentation/breaking-changes/)

## Next lesson

Continue with [Framework integrations: Angular, React, and Vite](045-framework-integrations-angular-react-vite.md).
